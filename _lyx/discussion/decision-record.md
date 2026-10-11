# Decision record — pane-inject-safety

## Goal

Text lyx types into a Claude pane never merges with, submits or wipes an operator's draft, and never closes a menu the operator has open.
A draft in a parked driver's pane delays batten's resume instead of failing batten, and a batten run that ends before its teardown while its child loom run is done is flagged and closed by the orch's after-landing routine.

## Scope

In:

- shuttle's verified send (`sendVerified`, the idle wait, the settle and submit confirmation), the start prompt, the skill-load message, every gate send and every caller of `Runner.Send`.
- shuttle's `ClearSession`, `CompactSession`, `ReloadPlugins` and `TypeColor`, and start's color step.
- reed's session-name repair (`/rename` typed by the per-hub watchdog in `internal/reedengine/namerepair.go`).
- reed: a new atomic paste-submit transport op and a new operator-activity query; a new `reed.yaml` key.
- claudeengine: its typing sequences lose the leading Escape and the 300 ms settle; it declares that it takes the paste path.
- `judgeReadiness`: an unreadable box is held for an engine that reads its box.
- orchengine's watcher: a deferred typed step is a wait, not a tick failure.
- loom start's driver-resume branch: a new refusal kind for a busy or drafted driver pane, with its refusal-spec row, and a resume report path fixed once per park.
- battenshed: retries that refusal like `ErrChildNotParked`, and notices the orch once per stall.
- batten's status verb: a flag when batten has halted while its child loom run is done.
- the orch role stencil's `## After landing` section.
- PATTERN: one new entry for pane text injection.

Out:

- `Runner.Interrupt`, `Runner.Inject` and the engines' `InterruptSequence` and `ModelSwitchSequence`: explicit key deliveries that deliberately bypass idleness.
- File-based notice delivery to the orch through a Stop hook (board: "Later, not in this task").
- A persistent injection queue owned by shuttle or reed; creel (`creel` board entry) will own queues.
- The trust-prompt dismissal and other startup key choreography in `awaitStartup`.

## Decisions

### D1 — Layering: reed writes, shuttle decides

Decision: reed gains two transport ops, an atomic paste-submit and an operator-activity query, and keeps knowing nothing of providers.
shuttle decides when a write may happen (readiness, activity, box state) and confirms it landed.
reed's session-name repair, which cannot call shuttle, uses the same two reed ops beside its existing `SessionNamer` idle check.
Rationale: matches the layering the `creel` entry fixes (`creel → shuttle → reed → tmux`: reed does keystrokes, shuttle knows how text reaches the LLM safely), so creel later wraps this path instead of replacing it.

### D2 — Atomic paste-submit in one tmux invocation

Decision: reed's paste-submit runs one tmux invocation whose command list fills a uniquely named buffer (`set-buffer -b lyx-inject-<random> -- <text>` or `load-buffer -b … -`, plan's choice), pastes it with `paste-buffer -p -d -b <name> -t <pane>`, and ends with `send-keys -t <pane> Enter`.
A variant without the trailing Enter exists for the fallback in D5.
It resolves the pane through `resolvePaneInThisSessionLocked` like every other transport op.
Rationale: the operator cannot interleave keys inside one tmux command list, so the merge window shrinks from about a second (Escape, 300 ms, typing, settle reads, Enter) to the gap between the pre-write check and this one invocation.
A named buffer deleted by `-d` never touches the operator's automatic paste-buffer stack.
`-p` sends bracketed paste when the pane's application has enabled bracketed-paste mode (A7), so Claude Code sees an explicit paste end before the Enter.

### D3 — Pre-write guard: readiness, operator activity, empty box

Decision: every text write into an agent pane is preceded, immediately before the reed call, by one fresh reading that must pass all of:

- the readiness reading (`sessionReadiness`) is ready, or, when unknown, the pane fallback reads idle;
- no attached tmux client whose active pane is the target pane has `#{client_activity}` newer than the `reed.yaml` key `operator_quiet_s` (default 5 s);
- the input box reads empty; for an engine that implements `InputBoxReader`, a capture with no box fails this on both paths, through D4 on a known reading and through the engine's idle reading, which requires an on-screen empty box, on the unknown fallback.

A failed reading holds the write; the idle wait (`awaitIdleSession`, bounded by `send_ready_timeout_s` and the send deadline) keeps polling, and when it runs out the send fails with `ErrSessionBusy` naming the reason, with nothing typed.
The operator-activity hold has its own reason text ("the operator typed in this pane within the last N s").
Bound: an operator-activity query that errors is logged at Warn and does not hold the write, so a tmux that cannot answer `list-clients` cannot stall every run; the empty-box check and D5's Enter guard still stand.
Rationale: preventing the merge is the only fix that protects the draft; a later check only limits the damage.
5 s covers ordinary mid-sentence pauses and costs an injection at most 5 s after the operator's last key.

### D4 — An unreadable box is held

Decision: `judgeReadiness` holds a reading whose box was not found ("no input box is on screen; a menu or picker may be open") for an engine that implements `InputBoxReader`.
An engine without `InputBoxReader` keeps today's behavior (no box ever read, not held on that ground).
Rationale: a picker or menu shows no box, and today's ready reading lets the leading Escape close it.
Bound: limited to box-reading engines, so a box-less engine is not held forever.

### D5 — Last guard: an Enter only on an exact match

Decision: every Enter lyx sends on its own, after the atomic paste-submit, goes out only when a capture taken just before it shows the box holding exactly the sent text (normalized by `normalizePaneText`) or exactly Claude Code's collapsed-paste placeholder.
The 48-character needle match in `boxHoldsSentText` and the multi-copy and prefix acceptance in `isOwnText` are replaced by that exact match.
After the paste-submit, confirmation is: a turn start past the send's events offset, or an empty box, means landed;
an exact match means the Enter did not take, so a guarded Enter is sent and the read repeats within `submit_confirm_timeout_s`;
a capture with no readable box (a menu a pasted slash command opened, a redraw) sends no Enter and the read repeats within the same window, and a window that ends that way fails as not landed (`ErrSubmissionNotLanded`), with no Enter and no clear key sent.
Any other box content means the operator's text is in the box, so no Enter, no clear and no edit is sent, and the send fails in one of two ways, both wrapping `ErrSessionBusy` so every retrying caller keeps working (D7):

- the box does not contain the sent text: nothing of ours is in it, and the failure is a plain hold that the next pre-write check holds until the box reads empty;
- the box contains the sent text beside other text: our text sits in the operator's draft, and the failure is its own sentinel (`ErrSentTextInDraft`, plan names it).

Both outcomes after the paste-submit that may leave our text in the box, the in-draft failure and the not-landed failure, record the pending send (its text and events offset) in the strand's run state; a confirmed landing and a pre-write hold record nothing.
The next send of the same text to that strand, once its pre-write guard passes on an empty box, returns landed without typing when a turn start past the recorded offset exists (the text went out with the draft or with an Enter the operator pressed), and types afresh otherwise (the operator cleared it); either way the record is cleared.
A slash command starts no turn, so its record always types afresh: a pending `/clear`, `/compact`, `/reload-plugins`, `/color` or `/rename` the operator submitted is typed a second time, which repeats a session command whose second run changes nothing but, for `/compact`, costs one more summary turn (R8).

`clearUnlandedText` clears only a box that holds exactly this send's text, re-read immediately before the clear key.
If the live check (Acceptance) shows Claude Code does not submit on an Enter right after a paste end, the engine switches to the no-Enter variant of D2 and the first Enter becomes a guarded one; the confirmation above is unchanged.
Bound: the placeholder acceptance can submit an operator's own long paste that replaced a swallowed paste of ours.
It is limited to a sent text long enough that the engine collapses it (claudeengine owns the threshold, which the live check pins), to a placeholder whose line count, when it shows one, equals the sent text's, to a box that read empty before the write (D3), to the send's own `submit_confirm_timeout_s` window, and to a fresh operator-activity check (D3's query) passing immediately before each guarded Enter.
That check covers only the last `operator_quiet_s` before each Enter, not the whole window: an operator paste older than that but inside the window passes it (R3).
The pending-send record can drop a retry wrongly when the operator deletes our text from the draft and sends the rest; the turn start then looks like ours (R6).
Rationale: the operator's draft is never submitted or edited by lyx; an Enter that Claude Code turned into a newline normalizes to an exact match and is recovered by one guarded Enter.
The pending-send record keeps a retry from typing our text a second time after it went out inside the operator's draft or after a not-landed window.

### D6 — One path for every injection

Decision: the start prompt, skill-load message, `Runner.Send`, gate sends, `ClearSession`, `CompactSession`, `ReloadPlugins`, `TypeColor` and start's color step all go through D3's guard, D2's paste-submit and D5's confirmation.
`ClearSession`, `CompactSession` and `ReloadPlugins` no longer type on the caller's earlier probe.
claudeengine's `ComposeSend`, `TypeSequence`, color and session-command sequences lose the leading Escape and the `composeSendSettleMS` pause; the engine declares that it takes the paste path, and an engine that does not keeps `playInputs` behind the same pre-write guard.
reed's name repair checks operator activity and the `SessionNamer` idle reading, then uses the paste-submit; it keeps retrying on the next watchdog tick when held.
Confirmation for slash commands, which start no turn, is the box reading empty, as `typeColor` reads it today.
Rationale: the board names every one of these as a merge source; one path means one set of guards to test.
Prompt suggestion is off in every shuttle-launched session (`promptSuggestionEnabled: false`), so the Escape that cleared it guards nothing and can complete a double-Esc or interrupt a turn.

### D7 — Callers treat a held write as "later", never as a failure

Decision:

- orchengine's watcher treats `ErrSessionBusy`, including both D5 foreign-box failures, from any typed step (notice, handoff request, clear, compact, reload, skill load, color, resume pointer) as nothing typed: the phase and the notice stay as they are and the next tick retries the same text; it does not count toward `maxConsecutiveTickErrors`.
- Gate sends and seat notices already retry `ErrSessionBusy`; both D5 foreign-box failures wrap it, so those retries keep working.
- `lyx shuttle send` reports the held error with its reason and leaves the retry to its caller; a rerun with the same text is deduplicated by D5's pending-send record.
- shuttle's `Start` keeps a held or foreign-box failure of its skill load or prompt delivery a died start, as `ErrSessionBusy` is today, and the re-step respawns the run.
  A pre-write hold dies only after waiting out `send_ready_timeout_s`; a foreign-box failure comes after the paste-submit and dies at once, killing the pane with the operator's draft in it.

Bound: the watcher's existing `HandoffTimeout` still marks a cycle stuck, so a draft left forever surfaces in `State.Stuck` rather than retrying unseen.
A held step outside a cycle, such as an idle-phase notice, records its hold reason in `State.Stuck` under its own marker beside `StuckByHold`, which the watcher's passing idle probe leaves alone and only a send that passes clears, so a notice held by the operator's activity or draft stays visible in `lyx orch status` instead of flapping each tick.
A died start kills its pane, so an operator who drafts in a pane that has not received its prompt yet loses that draft with the pane, after `send_ready_timeout_s` on a hold and at once on a foreign-box failure (R7).
Rationale: the board's "waits in a queue and is retried, never dropped"; every caller but `Start` and `lyx shuttle send` already has a queue or a retry loop (notice files, cycle phases, the gate's unsent re-prompt, seat notices, batten's resume retry), so no new queue module is built.
`Start` has no caller that could wait on its delivery without holding a half-started run, and a draft in a pane no prompt has reached yet is rare.

### D8 — Loom start refuses a busy driver pane with its own kind

Decision: when the resume branch's send (`resumeParkedDriver`) fails with `ErrSessionBusy`, which covers both D5 foreign-box failures, `lyx loom start` and `lyx loom resume` refuse with a new kind (`driver_busy`, declared in `internal/shedrun` beside `StartNotParkedKind`), whose message names the reason from shuttle (including the draft text) and the way forward: clear or send the draft in the driver's pane, then rerun the same verb, or `lyx batten run <slug>` from the prime.
The park marker stays in place, as today.
A never-appeared `ErrSubmissionNotLanded` keeps today's non-retryable refusal, since it does not wrap `ErrSessionBusy`.
The resume branch picks the line's report path once per park: the first attempt stores it in the run's scratch directory beside the park marker (path declared in `internal/shedrun`), every rerun while the marker stays reuses it, and it is removed with the marker, so each rerun types the same line and D5's pending-send record matches it.
A resume line that went out inside the operator's draft resumes the driver, whose own re-step removes the park marker, so batten's next attempt takes no resume branch and types nothing twice; the pending-send record covers a rerun that reaches the send first.
A row is added to `contracts/specs/refusal-spec.md` and a test reaches it (`PATTERN-refusal-way-forward`).
Rationale: batten tells refusals apart by kind across the process boundary (`childNotParked` in `internal/battencli/wire.go`), so a kind is the one signal batten can retry on.

### D9 — Batten retries a busy driver and notices the orch once

Decision: `childSpawnError` maps the `driver_busy` kind to a new battenshed sentinel, and `innerRunProducer` retries it exactly as it retries `ErrChildNotParked`: at most once per `noticeProbe`, the decision left unacted, its report naming the draft and the remedy ("clear the draft in the driver's pane; batten resumes it").
On the first busy refusal of a stall batten queues one orch notice through its existing notice path, and none again until a resume succeeds.
The notice names the slug, the stall and the remedy and never quotes the draft, so no text from another pane is typed into the orch session.
Bound: the retry has no deadline, as the not-parked retry has none; it ends when the resume lands, the run is paused or sent elsewhere, or the batten run is stopped, and the notice plus `lyx batten status` make the stall visible.
Rationale: the operator's draft is transient and theirs to clear; halting batten would recreate the teardown fault this task fixes.

### D10 — Batten status flags a halted batten over a done child

Decision: batten's `StatusExtras` hook adds a key when the batten run is not running and not done (failed, blocked or paused) and its child loom run in the pair reads done: it names the stall and the step that closes it, `lyx batten run <slug>`, which runs Worktree-Teardown.
No key is added when the pair or the child status is gone.
Rationale: the board's "flagged" requirement; every halted state has the same remedy, so the flag does not single out `failed`.

### D11 — The orch's after-landing routine closes batten

Decision: the orch role stencil's `## After landing` section gains a first step: read `lyx batten status <slug>`; when it is not done, run `lyx batten run <slug>` and confirm it reaches done.
Rationale: the board's third batten bullet; it catches a batten that died for any reason, not only a draft.

### D12 — PATTERN entry for pane text

Decision: add `PATTERN-pane-text-injection` under "Agents and prompts": typing text into an agent pane goes only through reed's atomic paste-submit behind shuttle's pre-write guard (reed's name repair behind the same activity check), and no Enter goes into a box that does not hold exactly the sent text.
Rationale: a new cross-cutting invariant goes into PATTERN in the same commit (CLAUDE.md); without it a future caller can reintroduce `send-keys -l` plus a blind Enter.

## Constraints

- `PATTERN-orch-pane-single-writer`: the orch watcher stays the only Go writer into the orch pane; D7 changes only how it reacts to a held write.
- `PATTERN-shuttle-provider-seam`: Claude-specific facts (no Escape, paste path declared, placeholder shape) stay in `internal/shuttleengine/claudeengine`.
- `PATTERN-shell-mechanics-seam`: reed passes the tmux command list as exec arguments, never as a pane-shell string; no `internal/shell` change.
- `PATTERN-refusal-way-forward` and `PATTERN-no-denied-recovery`: D8's refusal names a way forward the agents may run.
- `PATTERN-stencil-ownership` and `PATTERN-parent-directive`: D11 edits the committed orch stencil, read at call time; it does not tell the orch to ask the operator.
- `PATTERN-daemon-wakeups`: D9's retry keeps `noticeProbe` pacing; the activity hold adds no new unbounded wait.
- `PATTERN-config-strictness` and `PATTERN-binary-change-reconcile`: `operator_quiet_s` is a template key of the existing `reed` config module; an older `reed.yaml` takes the default until reconcile writes it.
- `PATTERN-spawn-observability`: two new tmux invocations are spawned, the activity query on every pre-write reading (each idle-wait poll and each guarded Enter included) and the paste-submit on every write, and both are logged through `internal/logger`.
- `PATTERN-test-speed` and `PATTERN-test-isolation`: shuttle and orch tests run against fakes untagged; reed's paste-submit and activity query get tmux-tier tests through `tmuxkit.Main`; the Claude live check is `llm`-tier.
- `PATTERN-testkit`: fakes for the new reed ops are added to the existing kits (`shuttlefake` and reed's fakes), not duplicated.
- Docs in the same commit: `doc.go` of `shuttleengine`, `claudeengine`, `reedengine`, `battenshed`, `loomcli` and `orchengine` where behavior changes; `template.yaml` comments for `submit_settle_ms` (no longer the slash-command Enter pause) and the new `reed.yaml` key.
- Test budget: a package pushed past its row in `cmd/lyx/testdata/test-budget.yaml` lists that file under `Edit:`.

## Auto-mode assumptions

- A1: The operator's 2026-10-11 board text is the spec; no operator was available, and every pick below its "Wanted" lists is mine.
- A2: `operator_quiet_s` defaults to 5 s (board: "a few seconds").
- A3: Retry (not `blocked`) is batten's response to a drafted driver pane, plus one orch notice per stall.
- A4: An operator-activity query error fails open with a Warn.
- A5: The D10 flag covers failed, blocked and paused, not only failed.
- A6: `#{pane_id}` in a `list-clients -t <session>` format resolves to each client's active pane of its current window; the plan verifies this in the tmux-tier test.
- A7: Claude Code enables bracketed-paste mode, so `paste-buffer -p` delivers a bracketed paste and a long paste collapses into the placeholder; the `llm`-tier live check confirms it.

## Open risks

- R1: Claude Code may treat an Enter arriving right after a bracketed paste end as a newline, or may open the slash-command menu on a pasted `/clear` and let the Enter pick another entry. D5's guarded Enter recovers the first; the `llm`-tier live check decides whether the engine uses the no-Enter variant, and the second is the same today with typed text.
- R2: A key the operator presses in the gap between the pre-write capture and the tmux invocation (milliseconds) can still precede the paste and be submitted with it; D3's activity hold makes this need a first keystroke after at least 5 s of quiet landing inside that gap.
- R3: A long text collapses into the paste placeholder, which an operator's own paste also produces; within D5's bound, a placeholder seen right after our write is ours with high likelihood, not certainty, and an operator paste more than `operator_quiet_s` before a guarded Enter but inside the confirm window passes the activity check.
- R4: `client_activity` also moves on mouse and resize events, so scrolling a watched pane delays injection by up to 5 s; accepted.
- R5: D9's retry has no deadline; a draft left for hours stalls the run, visible only through the notice and `lyx batten status`.
- R6: Our text left inside the operator's draft (D5's in-draft failure) goes out with the draft if the operator sends it, or is lost if they clear it; D5's pending-send record decides the retry by a turn start past the send's offset, which misreads an operator who deletes our text and sends the rest, dropping that one retry.
- R7: A start whose skill load or prompt delivery stays held past `send_ready_timeout_s`, or fails on a foreign box after the paste-submit, dies, and its pane goes with any draft the operator typed into it (D7).
- R8: A slash command left pending by D5 and then submitted by the operator is typed again on retry, since no turn start tells it landed; the session commands lyx types change nothing on a second run, and a second `/compact` costs one summary turn.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-2-review.md; fixer report: _lyx/reviews/discussion/round-2-fixer-report.md.
  - d8-dedupe-resume-line-unstable (class design, severity MEDIUM, open in rounds 2)
  - not-landed-may-have-landed-no-record (class design, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Discussion-Review -->

<!-- lyx:carry-over Plan-Review -->
- Plan-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/plan/round-1-review.md; fixer report: _lyx/reviews/plan/round-1-fixer-report.md.
  - card10-unexported-constant-from-shuttlecli (class consistency, severity MEDIUM, open in rounds 1)
  - card6-foreign-box-error-omits-draft (class consistency, severity MEDIUM, open in rounds 1)
<!-- /lyx:carry-over Plan-Review -->

## Acceptance criteria

- With the operator typing in the target pane (a fake attached client with recent activity), a send, clear, compact, reload, color and skill load types nothing and is held with the operator-activity reason; once activity is older than `operator_quiet_s` it goes through.
- No attached client, or a client on another pane, never holds a write.
- An activity query that errors logs at Warn and does not hold the write; the empty-box check still holds a write into a drafted box.
- A box-reading engine whose capture shows no input box is held by `judgeReadiness`; an engine without `InputBoxReader` is not held on that ground.
- A write issues one reed paste-submit, with no Escape and no typed text keys before it.
- reed's paste-submit, against a real tmux pane, delivers the text and the Enter through a named buffer that no longer exists afterwards and leaves the default buffer stack untouched (tmux tier).
- reed's activity query reads a recently active attached client on the target pane as active and one on another pane as not (tmux tier).
- After a paste-submit, a box holding exactly the sent text or the placeholder gets a guarded Enter, and none while the operator-activity check fails; a box holding the sent text plus other characters, or other text, gets no Enter, no clear key and no edit, and the send fails with an error wrapping `ErrSessionBusy`; a capture with no readable box gets no Enter and, at the window's end, fails as not landed.
- A send that failed with the sent text in the operator's draft, or as not landed after a no-box window, retried with the same text after the box reads empty, types nothing and returns landed when a turn start past its recorded offset exists, and types the text once otherwise.
- A placeholder in the box gets a guarded Enter only for a sent text long enough to collapse and, when the placeholder shows a line count, only when it equals the sent text's.
- A sent text of 48 or more characters is no longer accepted as held by a box that merely contains its first 48 characters.
- `ClearSession`, `CompactSession` and `ReloadPlugins` refuse to type into a box holding a draft even when the caller's earlier probe was idle.
- reed's name repair types nothing while the operator is active in the pane, and uses the paste-submit when it types.
- The orch watcher, given a held or foreign-box error from a typed step, keeps its phase and notice, retries on the next tick, and does not count the tick as failed; a held idle-phase notice records its hold reason in `State.Stuck`, a passing idle probe on the next tick leaves it, and a later send that passes clears it.
- shuttle's `Start`, given a skill load or prompt delivery held past `send_ready_timeout_s`, or one that fails on a foreign box, returns a died start whose error names the cause.
- `lyx loom start` and `lyx loom resume`, resuming a parked driver whose pane holds a draft, refuse with kind `driver_busy`, name the draft and the way forward, and leave the park marker; a rerun against the same park types the same resume line; a never-appeared send keeps its non-retryable refusal; the refusal-spec row exists.
- Batten given a `driver_busy` child refusal stays running, retries at the `noticeProbe` cadence, reports the draft and remedy, and queues exactly one orch notice per stall, which does not quote the draft.
- `lyx batten status <slug>` for a failed, blocked or paused batten run whose child loom run is done carries the flag naming `lyx batten run <slug>`; a running or done batten run, or one with no child status, does not.
- The orch role stencil's after-landing section names the `lyx batten status <slug>` check and the `lyx batten run <slug>` close.
- An `llm`-tier test pins whether Claude Code submits a plain text and a slash command (`/color`) on an Enter right after a bracketed paste end, and that a long paste collapses into the placeholder (A7); the operator runs it, and claudeengine's choice of D2 variant matches its result.
- `PATTERN.md` carries `PATTERN-pane-text-injection`.

## Notes for the plan writer

- The current typing path: `sendVerified` and `boxHoldsSentText` in `internal/shuttleengine/run.go`; `awaitSettledBox`, `confirmSubmitted`, `clearUnlandedText` and `isOwnText` in `send.go`; `judgeReadiness` in `readiness.go`; the session commands in `session.go`; Claude's sequences in `claudeengine/startup.go`, `session.go` and `color.go`.
- reed's transport ops are in `internal/reedengine/io.go`; client queries already exist in `attach.go` (`list-clients`); name repair is `namerepair.go`.
- The orch watcher's tick-failure counting is in `internal/orchengine/watchloop.go`; its typed steps are in `watcher.go`.
- Batten's not-parked plumbing: `childSpawnError` and `childNotParked` in `internal/battencli/wire.go`, the retry in `internal/battenshed/innerrun.go`, notices in `internal/battenshed/notice.go`.
- `awaitSettledBox` and `submit_settle_ms` exist to keep an Enter out of a typing burst; with a bracketed paste the plan decides whether the settle stays for the no-Enter variant only.
