# Decision record — orch-cycle-hardening

## Goal

`lyx orch` cycles its context on a fresh reading, by `/compact` or `/clear` as configured, and the hub runs under it for days without hand repair.
The orch keeps no scratch watcher: batten tells it about every run change it needs, through the orch watcher, the one Go writer into the orch pane.

## Scope

In:

- A context reading that follows compaction, manual or automatic, and is re-read before any automatic cycle fires.
- `cycle_mode: clear|compact` in `orch.yaml`, default `compact`, with a compact cycle that writes no handoff file.
- `permission_mode` accepts `bypass` only.
- Batten run notices to the orch (`orch-no-watcher`): state changes, a dead driver strand, a quiet run; delivered by the orch watcher.
- Stencil changes: a compact stencil, handoff, resume, adopt and start stencils without watcher re-arming.
- The hardening list: stale reading after adopt, PID reuse at adopt, transcript read from the end, the smoke bypass check, a large-session adopt smoke test, the orch pane's height on attach.
- Docs: `internal/orchengine/doc.go`, `internal/orchcli/doc.go`, `internal/battenshed/doc.go`, the `orch.yaml` template comments, `PATTERN.md` (one new entry).

Out:

- reed's `window-size latest` policy (see the attach decision).
- `orch-owns-the-loop`'s detached batten start, land command, `--continue` and role stencil.
- `stencil-loads-skills`, `batten-windows`, `runs-overview`.

## Decisions

### Context reading

- **D1. A compaction boundary is a reading.**
  The provider's context reader takes, of the latest main-chain assistant usage entry and the latest compaction-boundary entry, whichever is later in the transcript; a boundary yields its post-compaction token count.
  The reading also reports whether it came from a boundary, and that boundary's timestamp.
  Rationale: the 2026-10-04 incident cycled a 14k-token session on a 354k reading, and a boundary is the only transcript record a `/compact` leaves before the next turn.
  The transcript shape stays in `internal/shuttleengine/claudeengine` (PATTERN-shuttle-provider-seam); the shuttle seam carries a provider-neutral reading.
- **D2. Re-read before firing.**
  In phase idle, once a hard or soft trigger has passed every gate, the watcher re-reads the context through the newest turn end it holds, saves the new reading, and fires only if the re-read reading still meets that trigger's threshold.
  A requested cycle does not depend on the reading and is not re-read.
  Bound: a requested cycle is operator-initiated, skips only the threshold check, still needs phase idle and the idle probe, and so can cycle a small or fresh session on the operator's say; it is not subject to the D8 hold.
  Rationale: a turn-end reading goes stale whenever the transcript changes without a turn end (manual `/compact`, auto-compaction); one read per firing attempt is cheap.
- **D3. Read the transcript from the end.**
  The reader reads backwards in growing chunks until it has found the latest qualifying entry of each kind or reached the file start, so a typical read touches the tail only.
  Rationale: `ContextTokens` runs on every idle turn end and the hub transcript is hundreds of megabytes over days.
- **D4. A fresh launch or adopt resets the reading to unknown.**
  `ResetForFreshLaunch` clears `LastContextTokens` and `LastContextKnown`; `status` shows unknown until the new session's first turn end.
  Rationale: the previous session's reading says nothing about the adopted or relaunched one, and an unknown reading never triggers a cycle (existing rule).

### Cycle mode

- **D5. `cycle_mode: clear|compact`, default `compact`.**
  An empty or absent key means `compact`; any other value is a config load error naming both accepted values.
  Rationale: operator decision; compact keeps the session id and its Remote Control link.
- **D6. The compact cycle.**
  Its phases are idle and compacting.
  When a trigger fires, the watcher persists compacting, then types `/compact <focus>` rendered from a new one-line stencil `orch-template-compact.md`, whose focus text tells the summary what to keep: runs in flight with slug, state and attach command, open parent-review forks, operator requests and decisions pending, work half done.
  The phase ends when the reading reports a compaction boundary timestamped at or after the phase entry and the idle probe passes; that boundary's tokens become the reading and `CycleCount` increments.
  No handoff file is written, and `LastHandoff` is untouched.
  The typing rule holds: nothing is typed unless the idle probe passed on the same tick.
  Restart rule: a restarted watcher in compacting whose injection is unconfirmed re-types `/compact` only when the idle probe passes and no qualifying boundary has been read.
  Rationale: operator decision that the handoff instruction becomes the focus argument; the boundary is the only completion signal a `/compact` gives, since it produces no turn end.
- **D7. Compact cycles have no DEFER handshake.**
  The soft trigger in compact mode fires the compaction directly after `soft_idle_s`, with the same gates as today minus the DEFER exchange.
  Rationale: DEFER exists because `/clear` loses in-flight work; compaction keeps a summary, and background tasks are assumed to survive it with their completions still arriving (see Auto-mode assumptions and Open risks).
  Bound: if the survival assumption is false, a compact cycle loses a background task's in-flight completion with no handshake to catch it; `cycle_mode: clear` restores the DEFER handshake, and the compact smoke test asserts a completion arrives.
- **D8. A failed compaction holds the next one.**
  A compacting phase past `handoff_timeout_s` returns to idle with the abort reason `compaction timed out`, re-reads the reading, and records the abort time in `LastDeferral`, so no automatic trigger fires again for `soft_idle_s`.
  Bound: the hold delays the hard trigger too, by at most `soft_idle_s` per failed attempt; a requested cycle (`lyx orch cycle`) is not held.
  Rationale: without a hold, a compaction that keeps failing would be retried every idle grace.
- **D9. The clear cycle is kept unchanged** apart from the stencil edits in D14, and the four-phase machine and its guarantees stand for `cycle_mode: clear`.

### Permission mode

- **D10. `permission_mode` accepts `bypass` only.**
  The key stays in the template with value `bypass` and a comment saying it is the only accepted value and the orch never prompts per action; empty means `bypass`; any other value, `prompt` included, is a config load error that names the fix (`set permission_mode: bypass or remove the key`).
  Bound: the load stays `orchengine.LoadConfig`'s `configengine.LoadOrTemplate`: an absent file resolves the template (value `bypass`), a present file with another value errors, and both value checks (D5, D10) run in `LoadConfig` after the unmarshal.
  The error is raised wherever the orch command group loads its config, which is every `lyx orch` verb, so `start`, `status`, `cycle` and the rest all refuse with the fix named, before any pane is touched.
  A running watcher is unaffected until it restarts, so a hub on an old `orch.yaml` refuses to restart rather than running unprompted;
  the operator fixes the file before any verb, `status` included, answers again.
  The orchengine doc's operator-lever paragraph is removed.
  Rationale: operator decision; `prompt` passed no flag and fell back to the user's `defaultMode`, so it never meant what it said.

### Watchers and batten notices

- **D11. The orch runs no watcher of its own.**
  The start, resume and adopt stencils say: rely on driver messages and batten notices, never start a polling shell or Monitor for a run.
  The handoff stencils stop asking for a watcher list.
  Rationale: operator decision (`orch-no-watcher`); the 2026-10-02 smoke run showed background tasks survive `/clear`, and they are assumed to survive `/compact` (unverified, see Open risks), so cycles never needed re-arming; only an adopt or reboot (a new process) kills them, and with no watchers there is nothing to re-arm.
- **D12. Batten writes run notices.**
  `lyx batten run`'s Run-Shed row, which already polls the child's `status.json`, writes one notice per episode for:
  a change of the child's state away from running (awaiting, blocked, paused, failed, done);
  the driver strand found dead while the child is running;
  the child's `status.json` unchanged for `notice_quiet_min` (batten config, default 45) while the child's state is running and the driver strand is alive.
  An episode ends when the child returns to running or the status file changes, so each condition notifies once per episode.
  Episode state is persisted beside the run's other batten state, so a batten restart does not re-notify an episode already notified.
  A notice is one line starting with the literal prefix `[batten notice]`, naming the slug, the condition, the child state and its error if any, and the `cd <pair> && lyx reed attach` command.
  The error text is rendered on that one line: newlines and control characters become spaces and it is cut to 200 characters, so a notice never begins with `/` and carries no second line.
  Bound: notices are informational; batten's own control flow, budgets and halts are unchanged by them, and the orch acts on one only through `lyx batten status <slug>`.
  Rationale: closes the cases `internal/battenshed/doc.go` leaves out by design (dead, parked or stopped driver) in lyx code, as the operator decided.
- **D13. Notices reach the orch through the orch watcher only.**
  Batten appends a notice to a queue under the prime's `.lyx/orch/` (declared in `internal/orchcli/paths.go`) only when the prime's orch state records a strand; otherwise it only logs the notice.
  The orch watcher delivers queued notices oldest first, one per tick, as a typed turn, only in phase idle, only when the idle probe passes on the same tick, and removes each once typed.
  Queued notices wait while the watcher is down or a cycle is running.
  The queue is one file per notice, written atomically under a name that sorts by arrival, so concurrent batten processes never share a file.
  The queue holds at most 50 notices, checked by the appender: past that, the oldest is dropped and the drop is logged; concurrent appenders may overshoot the cap transiently, by at most one notice each.
  Delivery is at least once: a watcher that dies between typing a notice and removing its file types it again after restart.
  The watcher drops the queue at start when the orch state records no strand, and batten stops queuing when the strand is cleared; a recorded strand that is dead is the watcher's own dead-session case, so a notice for it waits at most until the watcher ends.
  Bound: a notice can be delayed by a busy session or a cycle, never typed over a draft or running turn; the queue grows only by one notice per episode per run and never durably past its cap.
  Rationale: one Go writer into the orch pane keeps the existing idle-typing guarantee; batten typing directly would race the watcher's own injections.
  This is a new cross-cutting invariant and goes into `PATTERN.md` in the same commit (`PATTERN-orch-pane-single-writer`: typing into the orch session from Go happens only in the orch watcher, idle-gated; other modules queue a notice).
- **D14. Stencils.**
  Handoff stencils (`orch-template-handoff.md`, `orch-template-handoff-soft.md`) drop the watcher list.
  The resume stencil drops "check each watcher it lists before re-arming it".
  The adopt stencil adds that background shells and Monitors of the previous process are gone and none is re-armed.
  The start stencil adds the notice rule: a line starting with `[batten notice]` (D12) is a batten notice; check the run with `lyx batten status <slug>` and act on it.
  `internal/orchcli/doc.go`'s run-loop step 2 is rewritten to the same rule.

### Adopt and attach hardening

- **D15. PID reuse at adopt.**
  A registry entry naming the session refuses the adopt only when its pid is alive and that process's start time matches the entry's recorded process start; where the live process's start time or the entry's recorded start cannot be read, today's behavior holds (a live pid refuses, and the message says the pid could not be proven reused).
  Bound: the check lets an adopt through only when a positive start-time mismatch proves the pid was reused.
  Rationale: a reused pid refused adopts falsely; failing closed where the start time is unreadable keeps two processes from driving one session.
- **D16. The orch pane's height on attach.**
  reed's `window-size latest` stays: `internal/reedengine/doc.go` records why it is not changed or conditioned, and the window returns to the larger client's size at that client's next keystroke.
  The orch side changes: when the idle probe fails because the pane is too short to draw an input box, the watcher records that as `State.Stuck` (`orch pane too short for the idle probe; resize or use the larger client`), so `lyx orch status` reports why cycles and notices are held.
  `State.Stuck` thereby also carries an idle-phase hold reason, set and cleared only by the idle probe's result, so the transitions that already reset it do not hide it.
  The reason is cleared on the next tick whose idle probe passes; notices keep queuing under D13's cap meanwhile.
  Rationale: the harm on a shorter client is a silently held cycle, which the status names; changing reed's window policy would break the attach chain for every session.

### Tests

- **D17. Smoke bypass check asserts the footer.**
  The smoke test asserts the bypass-permissions footer in the orch pane instead of running a tool the user's settings may already allow.
- **D18. Large-session adopt smoke test.**
  An opt-in smoke test, enabled by an environment variable naming a session id, copies that session's transcript into the test's sandbox project directory and adopts the copy, asserting startup within `startup_timeout_s` and no dialog left on the pane; it skips when the variable is unset.
  Bound: it reads only a copy and never touches the original session or its registry entry.
  The implementer runs it once against a real hub transcript and records the result under `internal/orchengine/doc.go`'s Open risks.

## Constraints

- PATTERN-shuttle-provider-seam: transcript and compaction-boundary shapes, and the `/compact` command text, live only in `internal/shuttleengine/claudeengine`.
- PATTERN-told-geometry: `orchengine` and `battenshed` take the notice queue path from their callers; the path is declared in `internal/orchcli/paths.go`.
- PATTERN-lyxdirs-single-declarer and PATTERN-durable-vs-ephemeral-state: the notice queue is ephemeral, under `.lyx/orch/`, spelled through `internal/lyxdirs`.
- PATTERN-stencil-ownership: the new compact stencil is read from the told stencils directory at call time.
- PATTERN-config-strictness: orch config keeps its current loader, `configengine.LoadOrTemplate`; the two new value checks are load errors raised in `orchengine.LoadConfig`.
- PATTERN-test-tier-purity, PATTERN-tmux-test-isolation: watcher, reader, config and notice tests are untagged with fakes; live checks are `smoke`.
- PATTERN-spawn-observability: no new process spawn is added.
- PATTERN-comment-line-breaks and the markdown rule: semantic line breaks in all doc edits.
- Stencils typed through Send render to one line.
- Docs land in the same commit as the code (CLAUDE.md), including the new PATTERN entry.
- Landing obligation: the `orch-owns-the-loop` board entry's "A watch command" section is covered by D12 and D13 and is amended at landing.

## Auto-mode assumptions

- Claude Code's `/compact` keeps the session id and transcript file and writes a compaction-boundary entry with a post-compaction token count, as the 2026-10-04 transcript showed.
- `/compact` accepts its focus as one line of free text.
- The Claude session registry entry carries a process start time usable to tell a reused pid apart.
- A background task started before a `/compact` still delivers its completion to the compacted session, as it does after `/clear`.
- The orch's batten runs are started from the prime, so the prime's orch is their parent.
- The idle probe can tell a pane too short to draw an input box from a busy one, from the pane's height against the rows an input box needs.

## Open risks

- The compaction-boundary entry is a Claude Code internal; if its shape changes, the reader degrades to the last assistant entry and a compact cycle times out instead of completing (D8 holds retries).
- A compaction summary may drop detail the handoff file would have kept; the focus text in D6 is the only lever.
- Reading the latest boundary by file order assumes Claude Code appends entries in order.
- The quiet notice can fire during a long legitimate step (a big Webster batch); it is informational and fires once per episode.
- If a background task's completion does not survive `/compact`, a compact cycle loses in-flight completions silently; the compact smoke test asserts a completion arrives, and D7's missing DEFER is the unbounded edge until it does.
- If the registry records no process start, D15 never lets an adopt through and the PID-reuse item ships with no effect; the refusal message says the pid could not be proven reused.

## Acceptance criteria

- Replaying the 2026-10-04 incident in a watcher unit test (reading 354815 at the last turn end, then a compaction boundary with 13673, soft threshold 300000) fires no cycle, and the saved reading is 13673.
- Reader unit tests: boundary after the last assistant entry yields the boundary's tokens and marks it compacted; an assistant entry after the boundary yields the assistant usage; sidechain and zero-usage entries are ignored; a qualifying entry spanning a chunk edge is read whole; a file with neither yields unknown.
- `ResetForFreshLaunch` leaves the reading unknown.
- Config tests: `cycle_mode` empty or absent is `compact`, `clear` and `compact` load, anything else is an error naming both; `permission_mode` empty or `bypass` loads, `prompt` and anything else is an error naming the fix.
- Compact-cycle watcher tests: a trigger types `/compact` with the rendered focus only on an idle-probe pass; the phase completes on a boundary at or after entry plus idle; an earlier boundary does not complete it; timeout returns to idle with `compaction timed out` and holds the next automatic trigger for `soft_idle_s` while a requested cycle is not held; a restart in compacting re-types only on idle with no qualifying boundary; no DEFER handling in compact mode.
- Clear-mode watcher tests pass unchanged apart from stencil text.
- Notice tests: each condition in D12 writes one notice per episode, the quiet condition only for a running child, and a batten restart does not re-notify; a notice renders as one line with the `[batten notice]` prefix, with newlines and control characters in the error replaced and the error cut to 200 characters; the queue caps at 50 and drops the oldest; `notice_quiet_min` defaults to 45 in batten's config; no orch strand recorded logs only; the watcher delivers oldest first, one per tick, only in phase idle on an idle-probe pass, and removes delivered notices; a non-idle probe or a running cycle leaves the queue untouched.
- PID-reuse tests: matching start time refuses; mismatching start time lets the adopt through; unreadable start time refuses.
- A too-short pane records the D16 stuck reason, `lyx orch status` shows it, and the reason clears on the next passing probe.
- A config with `permission_mode: prompt` fails every `lyx orch` verb, `status` included, and the watcher start, with the fix named, before any pane is touched.
- The rendered compact stencil is one line.
- Smoke: a live compact cycle keeps the session id and the strand, leaves a reading below the soft threshold, and a background task started before it still delivers its completion afterward; the bypass footer is asserted; the large-session adopt test runs when its variable is set and its result is in `internal/orchengine/doc.go`.
- `internal/orchengine/doc.go`, `internal/orchcli/doc.go`, `internal/battenshed/doc.go`, the `orch.yaml` template and `PATTERN.md` describe the shipped behavior; the auto-compaction and pre-clear-resume residuals are updated for compact mode.

## Notes for the plan writer

- The incident's mechanics are in `internal/orchengine/watcher.go` (`tick`'s per-turn-end `ContextTokens` call, `tickIdle`) and `internal/shuttleengine/claudeengine/usage.go` (`transcriptContextTokens`).
- `tickIdle` already requires the newest event to be a turn end, so D2's re-read always has a turn end to read through; a restarted watcher in compacting has none in memory, so the compacting phase needs the transcript reference persisted in State (an opaque copy of the last turn end's payload is one way).
- In compact mode reed's resume path replays the same session id, so the doc.go residual about rebirth resuming the pre-clear context applies to clear mode only.
- `internal/battencli/wire.go` already wires `DriverAlive` for the Run-Shed row; the notice writer is another seam there.
- `internal/shuttleengine/claudeengine/resume.go` holds the registry check D15 changes.
