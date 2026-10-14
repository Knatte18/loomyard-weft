# Decision record — tender

## Goal

A `tender` module holds hub-wide control over what runs: pause, resume, revive after a network outage, one status view of every live session, and the stale-binary listing.
`lyx pause` and `lyx resume` stay as shortcuts.
A pause no longer consumes the deadline of a GitHub call it holds (issue #606), and the long-lived lyx processes that can restart safely pick up a replaced binary on their own.

## Scope

In:

- A new cobra module `tender` with the verbs `pause`, `resume`, `revive` and `status`.
- A conditioned send in `internal/shuttleengine`, which `revive` uses: it refuses every hold at once and, after its landing, waits a bounded time for the session's turn start (D5).
- `lyx pause` and `lyx resume` as shortcuts in `internal/shortcutcli/shortcuts.yaml`; the top-level `pause` and `resume` root commands go.
- The pause-held deadline fix for GitHub calls, and the transient re-step of a GitHub deadline error pinned by a test (#606).
- Self-restart on a replaced binary for `lyx orch watch`, and removal of the lingering `lyx reed attach` process.
- A hub-wide listing of lyx processes running a replaced binary, inside `lyx tender status`.
- Docs, PATTERN background, test budget and sandbox coverage that the rename and the new verbs touch.

Out:

- Drain (finish every step in flight, then stop).
- A cap on concurrent runs; it stays an open point of `orch-split`, whose Go run driver is what starts runs.
- Restarting a process that is mid-work (`lyx batten run`, `lyx loom step --until-stop`, `lyx shed step --until-stop`, agent sessions): these are listed only, and their existing build-change handling is not touched.
- A restart verb; a process that can restart safely restarts itself (D8, the reed watchdog, the status panes), and the rest is listed only.
- Changing the reed watchdog's or the status panes' binary-change handling, which `reed-window-order` built.

## Decisions

### D1 — Module layout: `internal/tendercli` over `internal/tenderengine`

`internal/pausecli` is renamed `internal/tendercli` and `internal/pauseengine` is renamed `internal/tenderengine`; `internal/pauserecord` keeps its name and role.
`tendercli.Command()` mounts one root command, `tender`, whose subcommands are `pause [<slug>] [--grace <seconds>]`, `resume [<slug>]`, `revive [<slug>]` and `status`.
`InstallNetworkGate` moves with the CLI package, and the root's pre-run calls it from there.
Rationale: `lyx --help` then lists modules only, and `pauserecord` is a leaf imported widely whose name still describes what it holds.

### D2 — Shortcuts `pause` and `resume` only

`shortcuts.yaml` gains `pause: [tender, pause]` and `resume: [tender, resume]`.
The shortcut forwards the operator's slug and flags to the end of the argv, so `lyx pause <slug> --grace 10` keeps working.
Refusal texts and stencils that name `lyx resume` or `lyx pause` keep doing so: the shortcut is the stable operator spelling.
`revive` and `status` get no shortcut.
Rationale: the board names these two; the others are rarer and stay discoverable under `lyx tender`.

### D3 — `status` replaces `lyx pause --status`

`lyx tender status` reports, read-only:

- the live pause entries, the network operations in flight and the holds, as `lyx pause --status` does today;
- every live agent session in the hub: pair, strand name, session state with its cause and since time, and whether a live pause entry covers it;
  the prime's sessions are listed too, the orch's and any other live agent session in the prime;
- the stale-binary listing of D7.

`pause` loses its `--status` flag, with no alias, and every place that names `lyx pause --status` names `lyx tender status` instead, `tools/sandbox/SANDBOX-CORE-SUITE.md` and `docs/overview.md` included.
`status` runs from any worktree of the hub, and any caller may run it, agents included, since it writes nothing;
it shows an agent the hub's session states and the argv of hub lyx processes, nothing a same-user process listing does not already expose, and it reads no transcript or pane content.
Rationale: one place answers "what is running, waiting, stuck or stale"; `lyx batten overview` stays per run, `lyx reed status` per worktree.

### D4 — `pause` and `resume` behave as today

Their sequences, caller rules, inputs and envelopes are unchanged apart from the rename and D3.
Rationale: the task moves them; it does not redesign them.

### D5 — `revive [<slug>]`: one carry-on line to each session stalled on an API error

`revive` covers the hub for no slug and one pair for a slug, with the same caller rule as `pause`: the orch and an external caller may name any scope, and an agent only its own pair, where its own session is busy and only its siblings can be revived.
It enumerates the task pairs' sessions the way `pause` does and, for the hub scope, also reads the orch's session from the prime's orch state; no other prime session is revived, and the report lists none.
It acts on each session whose state reads idle-stalled with cause `api-error`:

- a session a live pause entry covers gets no line and is reported as paused, since `resume` gives it its line;
- the orch's session gets no line from tender: tender queues an `orchengine` notice, which the orch watcher types at the orch's next idle, and queues none while an undelivered revive notice is already queued; two revives run at once may each queue one, which the orch reads as a repeated carry-on line.
  The watcher drops a queued revive notice at delivery when the orch's state no longer reads idle-stalled with cause `api-error`, and the report says the notice is conditional and whether a watcher is live, so a queued notice with no watcher reads as such;
- every other session gets one fixed line, declared once in `tenderengine`, through a conditioned send that `shuttleengine` gains for it.
  A conditioned send names a session state and cause its ready reading must show, and refuses at once, with no idle wait, when the reading shows anything else or any hold the ready reading names: a draft in the input box, operator activity in the pane or an unmatched turn start.
  It also refuses at once, sending no key, when its ready reading shows the rewind picker or a lone stray character in the input box, and it never clears its own unlanded text: a send that does not land is reported as not landed, with its text left in the box.
  Otherwise it types and confirms the landing as today's guarded send does, then waits for the session's hook-derived state to read a turn start past the send (shuttle's `turn_start` signal past the events offset read before the send), for at most one submit window (`submit_confirm_timeout_s`) from the landing;
  a send that lands with no turn start seen within that wait is reported as landed with no turn start.
  Revive's conditioned send names idle-stalled with cause `api-error`;
  a session that reads busy or any other state at that reading is reported skipped and never waited for, and a send the guard refuses for another reason is reported with its reason and not retried.
  Bound: a conditioned send adds refusals to today's send, drops its picker close, stray-character clear and unlanded-text clear, and adds a wait after its landing, so it never types where today's send would not and plays no key beyond its typed line and submit.

Shuttle's engine send does not consult the pause record, so `revive` reads a session's coverage and runs its conditioned send, the turn-start wait included, while holding the pause record's write lock, the lock `pause` takes to add its entry, and skips a covered session.
`revive` holds that lock by running inside `pauserecord.Update` with a mutate that changes no entry, so `pauserecord` gains no call and keeps its import set, and the write at release prunes expired resumed entries as every record write does.
A pause entry is then either live at that read, and the session gets no line, or added only after the revived turn's start reads in the session's state or the turn-start wait ends.
When the turn start read within the wait, the pause's grace poll reads the session busy and its interrupt pass records it, so `resume` gives it its line.
Two cases stay unrecorded.
A send that lands with no turn start seen within the wait lets the pause take its entry when the wait ends; its grace poll and interrupt pass read the session idle and record nothing, and a turn start that arrives after that interrupt pass runs its turn under the live entry, uninterrupted and unrecorded, until the turn ends by itself.
Bound: `revive` reports such a session as sent with no turn start seen, and `lyx tender status` shows it busy under the live entry, so the operator can pause that slug again.
A revived turn that ends before the pause's poll, such as an API error recurring during the outage, reads idle to the pause, which records nothing, so `resume` gives it no line, and a later `revive` does.
Bound: the lock is held for one conditioned send at a time, and each send holds it for one ready reading before typing, then at most one submit window from the start of typing to the landing, then at most one submit window for the turn start.
The ready reading is a fixed number of pane captures and event-file reads, with no settle, since the send refuses a picker or stray character at once instead of clearing it, and no wait for a draft, operator activity or an unmatched turn start, each refused at that one reading.
Each pane capture inside that span waits on reed's per-worktree operation lock and a tmux call with no timeout of their own, so the stated windows bound the polling, not a hung reed or tmux call.
`pauserecord.Update` blocks on that lock with no timeout, so a concurrent `pause`, `resume` or orch watcher record write waits at most that hold; coverage reads take no lock, so the send's own reads do not wait on it.
The report lists each session it saw, with what it did: sent, sent with no turn start seen, not landed, queued for the orch, paused, refused with reason, or skipped because it does not read an API-error stall.
Bound: `revive` types one fixed line, only into a session whose state reads idle-stalled with cause `api-error` at the conditioned send's own ready reading, or, for the orch, at the notice's delivery, and only through the conditioned send or the orch notice queue; it sends no key, kills nothing and changes no live entry.
A second `revive` finds a revived session busy or done and sends nothing, and finds the orch's revive notice still queued and queues none.
Rationale: shuttle's automatic continue covers an API error across a machine sleep only; a network outage without a sleep today needs `lyx shuttle send` per session by hand.

### D6 — A GitHub call held by a pause gets its full deadline after release (#606)

`pauserecord.Network` reports how long it held the call.
`githubclient` drops the http client's own `Timeout` and applies its 30-second client timeout itself in the transport, starting after the hold, so the request context carries only the caller's deadline.
When a call was held, the transport runs it under that deadline moved later by the held time, with the caller's values.
Every cancellation of the caller's context that arrives before its own deadline passes still ends the call;
once that deadline has passed, only the moved deadline ends the call, so the extension is the longest a late cancellation can be ignored.
The hold stays blind to the caller's context, so a call whose context is cancelled during the hold, before its own deadline passes, ends with the cancellation at release;
once that deadline has passed during the hold, a later cancellation is not observable and only the moved deadline ends the call.
`pauserecord.Network` keeps its context-free signature.
The extension logic lives in `githubclient`; `pauserecord` only reports the held time and keeps its import set.
Bound: the extension equals the time the gate held that one request, so an unheld request keeps exactly its deadlines, and a caller issuing several requests under one deadline gets the extension for the held request only; `landingshed` derives one deadline per call.
A pause that starts after the gate admitted a call does not hold or extend it.
`gitexec` already holds a remote command before its deadline starts, and stays as it is.
A GitHub deadline error stays classified transient (`githubclient.IsTransient`); with the hold no longer eating the budget, such an error means the call's own budget ran out after release, and the loom drive rules re-step it once before the tackler judges it, so a first occurrence never halts the run `failed`.
`loomengine.ClassifyHalt` keeps labelling a `failed` halt `producer-hard-failure` once that re-step is spent.
A caller whose one context deadline spans several GitHub requests, such as a paginated list, derives a fresh deadline per request instead, so each request's own extension covers its hold.
Rationale: the hold sits inside the transport, under deadlines its callers set before the call, so the transport must own both deadlines to move them.

### D7 — Stale-binary listing

`tenderengine` lists the lyx processes of the hub whose executable was replaced: on Linux, a process whose `/proc/<pid>/exe` link names a deleted file whose base name, with the `(deleted)` suffix stripped, equals the base name of the running lyx's own executable, wherever that binary lives (the Go bin dir and `.dev-bin` alike), and whose working directory, with the `(deleted)` suffix stripped, is the hub directory or a path inside it by path-segment containment, a deleted working directory included.
Each listed process shows pid, start time and argv; the listing attributes no process to a pair.
On a platform with no such reading, the listing reports that it is unsupported, and the rest of `status` is unaffected.
The process reading is a seam told to the engine, implemented beside `internal/proc`.
Rationale: the operator needs to see what still runs an old binary after a deploy; the list makes a self-restart that did not happen visible.

### D8 — `lyx orch watch` re-execs itself on a replaced binary

The orch watcher calls `selfexec`'s check from its own poll and re-execs at a tick where no cycle step is in flight and no hub pause entry is live at all, so the persisted cycle state carries over and no in-memory pause handling, the handled set and the first-Esc marks included, is live to be lost.
An entry the old image settled during its grace is never handed to the new image, so the new image never interrupts the orch for it.
The watch lock does not survive the exec, so the new image takes it again through the watcher's existing acquire with retry.
Bound: in that gap `lyx orch start` may spawn a second watcher; both contend for the one lock and the loser exits as already running, so exactly one watcher remains, and a `refresh` or `distill` in the gap reports `watcher_live: false` and withdraws its request, as it does with no watcher;
a hub `pause` or a `revive` in the gap reports no live watcher, and the new image holds the cycle machine under a live hub entry from its first tick and interrupts a busy orch only for an entry whose grace end is still ahead;
an entry whose grace ended in the gap leaves the orch uninterrupted, as with no watcher, which the pause's no-live-watcher report already says.
Rationale: the watcher is a long-lived daemon whose state is on disk, so a restart in place is safe.

### D9 — `lyx reed attach` replaces itself with the tmux client on Unix

On Unix, `reed attach` execs the tmux attach argv in place of spawning it as a child, so no lyx process outlives the handover; on Windows it keeps today's child process.
The exec skips the root's post-run, so the logger is flushed and the exec logged before it happens (`PATTERN-spawn-observability`); the process's run directory is left to the next start's sweep, as for the status panes' re-exec.
Bound: nothing runs after attach in that process, so `reed attach` must be the last argv of any shortcut; the shortcut guard in `cmd/lyx` checks it, and `prime` already has it last.
Rationale: the lyx parent only waits on tmux, so removing it is simpler than restarting it, and an attach then never runs an old binary.

## Constraints

- `PATTERN-cli-cobra`: `tender` is one cobra subtree; its background's `pausecli` deviation line is replaced by `tendercli` using `tenderengine` with one root command.
  The interactive-handoff exception for `attach` already covers D9.
- `PATTERN-cliwire-sole-wiring`: `tendercli` keeps `pausecli`'s documented reason for resolving the hub through `lyxcwd` itself.
- `PATTERN-told-geometry`: `tenderengine` resolves no path and spawns nothing; the process reading, the session enumeration, the send and the notice queue are seams.
- `PATTERN-agent-name`: pair and role of a session or caller are parsed through `internal/agentname` only.
- `PATTERN-pane-text-injection`, `PATTERN-orch-pane-single-writer`: see D5.
  `creel` plans to own every delivery into a pane; revive reaches panes through the same send and notice seams `pause` and `resume` use, so creel's wiring replaces all three together.
- `PATTERN-no-denied-recovery`: every new refusal names a way forward an agent may run.
- `PATTERN-sandbox-coverage`: the sandbox suite exercises `tender` or excludes it with a reason, replacing the entries for `pause`.
- `PATTERN-daemon-wakeups`, `PATTERN-log-levels`: the watcher's binary check adds no wake of its own and logs repeated polls at Debug.
- `PATTERN-leaf-packages`: `selfexec` and `pauserecord` keep their import sets.
- `PATTERN-github-auth`: every GitHub client is built by `githubclient`, so dropping its http client `Timeout` in D6 covers every GitHub call.
- Docs in the same commit: `doc.go` of both renamed packages and of `shuttleengine`, whose doc states the conditioned send's at-once refusals and its turn-start wait, the module table in `docs/overview.md`, and `pattern/PATTERN-cli-cobra.md`.
- `cmd/lyx/testdata/test-budget.yaml`: the `pausecli` and `pauseengine` rows are renamed, and raised where the new verbs' tests need it.

## Auto-mode assumptions

Every decision D1–D9 and the Scope's Out list are the writer's own picks, made with no operator answer; the support log's question ledger names each.

## Open risks

None.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 4: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/discussion/round-4-review.md; fixer report: _lyx/reviews/discussion/round-4-fixer-report.md.
  - d5-conditioned-send-lock-hold-unbounded (class design, severity MEDIUM, open in rounds 3, 4)
  - d5-revive-pause-race (class design, severity MEDIUM, open in rounds 1, 2, 3, 4)
  - d5-revive-sends-keys (class design, severity MEDIUM, open in rounds 4)
<!-- /lyx:carry-over Discussion-Review -->

<!-- lyx:carry-over Plan-Review -->
- Plan-Review, round 3: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/plan/round-3-review.md; fixer report: _lyx/reviews/plan/round-3-fixer-report.md.
  - held-page-row-page1-30ms-slack (class design, severity MEDIUM, open in rounds 3)
  - paste-row-unreachable-apierr-parser (class consistency, severity MEDIUM, open in rounds 3)
<!-- /lyx:carry-over Plan-Review -->

## Acceptance criteria

- `lyx tender pause`, `resume`, `revive` and `status` exist; `lyx pause` and `lyx resume` run them with slug and flags forwarded; `lyx --help` lists no top-level `pause` or `resume` module.
- A GitHub call held by a pause longer than both its deadlines succeeds after resume when the server answers in time.
- An unheld GitHub call keeps its deadlines unchanged, and a call admitted before a pause starts is neither held nor extended.
- A held GitHub call whose caller's context is cancelled during the hold, before its original deadline passes, ends with the cancellation once the hold releases, and one cancelled after release, before its original deadline passes, ends at once.
- A several-request GitHub caller whose second request is held by a pause completes after resume when the server answers in time.
- A transient GitHub failure at a landing producer is re-stepped and does not halt the run `failed` on its first occurrence.
- `revive` sends its line to an idle-stalled API-error session, not to a busy, done or differently stalled one, not to a paused one, and queues a notice rather than typing for the orch; a guard refusal is reported.
- A session that turns busy between `revive`'s state read and its send gets no line once that turn ends.
- A queued orch revive notice whose orch recovered before delivery is dropped, not typed.
- A hub-scope `revive` reads the orch's state and no other prime session.
- A pause entry live at `revive`'s coverage read for a session keeps the line from that session.
- A pause added while a `revive` line is in flight takes its entry only after that session's turn start reads or its turn-start wait ends, and interrupts and records the session when its turn start read within the wait; a revive send whose turn start is not seen within the wait is reported as sent with no turn start.
- A conditioned send to a session with a draft in its input box, operator activity in its pane, an unmatched turn start, an open rewind picker or a lone stray character in its input box refuses at once, with no idle wait and no key sent.
- A second `revive` before the orch's notice is delivered queues no second notice; `revive` with no orch watcher live reports the notice as queued with no watcher.
- `revive` with nothing stalled reports an empty result and succeeds; a slug with no pair is refused naming the slugs that have one; an agent naming another slug is refused.
- `status` shows pause entries, holds, sessions with their states, and the stale listing; each session shows whether a live pause entry covers it; with no pause, no sessions or no stale process each part reads empty.
- `lyx tender pause --status` is refused as an unknown flag, and no doc or stencil names `lyx pause --status`.
- The stale listing names a process whose binary was replaced, under the Go bin dir and under `.dev-bin` alike, and omits one whose binary is current; on an unsupported platform it reports unsupported.
- The orch watcher's binary check fires at a point with no cycle step in flight and not mid-cycle, tested through `selfexec`'s exec seam, since a test binary never execs.
- A watcher re-exec racing `lyx orch start` leaves exactly one watcher.
- The watcher does not re-exec while any hub pause entry is live, handled or not.
- `reed attach` on Unix leaves no lyx process behind once tmux is attached, and a shortcut with an argv after `reed attach` fails the shortcut guard.

## Notes for the plan writer

- The hold is in `internal/githubclient/transport.go` (`authRT.RoundTrip`); `internal/gitexec/gitexec.go` `runCore` shows the hold-before-deadline shape for git.
- See `internal/shuttleengine/hold.go` for shuttle's sleep-only API-error continue.
- `selfexec` is already used by `internal/statuspane`.
- The orch watch lock and its acquire are in `internal/orchengine/watchloop.go`; `QueueNotice` is in `internal/orchengine`.
- `spawn-audit-log` plans a recorded launch path and a process sampler; D7's process reading and D9's exec are what it will later cover.
