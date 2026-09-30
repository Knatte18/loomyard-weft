# Decision record — hub orchestrator as a reed strand with automatic handoff

## Goal

Host the hub orchestrator as an interactive Claude session in a reed strand of the hub's prime worktree, and cycle its context automatically before it fills:
when context usage crosses a threshold and the session is idle, lyx has the session write a handoff, clears it, and resumes it from that handoff.
The operator keeps talking to the same pane throughout, through `lyx reed attach` or the hub VS Code workspace terminal.

## Scope

In:

- A new CLI module `lyx orch` (`internal/orchcli` + `internal/orchengine`) with verbs `start`, `status`, `cycle`, `stop`, and the detached daemon verb `watch`.
- Launching the orchestrator session through shuttle as an interactive run in the prime's reed session.
- A detached, single-instance watcher process that reads turn-end events and context usage, and runs the handoff → `/clear` → resume cycle.
- Provider-side additions in `internal/shuttleengine/claudeengine` behind the `Engine` seam: context-usage reading from the transcript, an input-box-empty probe, and the clear-session key choreography.
- A provider-invariant shuttle surface for reading a run's parsed events and for typing text into its pane, if the existing `Send`/`Inject` surface does not already cover it.
- Stencils for the start prompt, the handoff instruction and the resume prompt.
- An `orch` config file with a template, loaded degrading.
- Docs: `docs/overview.md` module table, `CONSTRAINTS.md` list updates, a roadmap entry.

Out:

- Adopting an already-running Claude session from the operator's own terminal.
- Any lyx-side cross-session messaging; Claude Code's native `ListAgents`/`SendMessage` is used as-is.
- Changes to `lyx ide spawn` or the hub workspace (no auto-attach task).
- Writing to `manifest/HANDOFF.md` or any other tracked file.
- Automatic relaunch after the orchestrator process dies.

## Decisions

### D1 — Module name and placement: `lyx orch`

**Decision:** New module `orch`: `internal/orchcli` (cobra subtree, `Command()`/`RunCLI`/`RunCLIIn`) and `internal/orchengine` (engine, told its paths).
**Rationale:** "orch" is the operator's own name for the session (`lyxhub:orch`), and "hub" already names geometry (`hubgeom`, `hubforge`, the `-LYXHUB` suffix), so `lyx hub orch` would overload it with a one-verb parent.

### D2 — Runs from the warp prime only

**Decision:** Every `lyx orch` verb resolves through `lyxcwd` and refuses unless cwd is the hub's warp prime, using the same prime check batten's pre-run uses (`fabricengine.RequireDrivableWorktree` plus the prime-name comparison).
**Rationale:** The hub orchestrator lives in the prime on `main`; one orchestrator per hub; a task worktree has its own ly-drive session and must never host this one.

### D3 — The session is a shuttle run in the prime's reed session

**Decision:** `lyx orch start` launches the session through `shuttleengine.Runner` as an interactive run (`Interactive: true`, operator present), in the prime's reed session, with a stable display name `orch` and `Focus: true`.
Its model and effort come from the orch config.
The watcher owns the session's lifetime instead of shuttle's `Wait`: the run never finishes on an output file.
**Rationale:** Reuses the whole proc → reed → shuttle stack (interactive tmux, never `claude -p`), its startup gates, its Stop-hook events file and its pane key choreography.

### D4 — The watcher is a detached `lyx orch watch` process

**Decision:** `lyx orch start` spawns `lyx orch watch` detached, from the executable path, logging the spawn per Live-Substrate Spawn Observability.
The watcher is single-instance per prime via a lock under the prime's `.lyx/orch/`.
It polls the run's events file and exits when reed no longer tracks the strand or its pane is dead, recording that in orch state.
**Rationale:** reed's per-hub watchdog sits below shuttle and must stay provider-blind, so it cannot host a loop that reads provider transcripts; a detached lyx process matches the go driver's existing spawn pattern.

### D5 — `lyx orch start` is idempotent

**Decision:**
- Live strand and live watcher: start nothing, then attach.
- Live strand, no watcher: spawn the watcher, then attach.
- Dead or absent strand: remove any corpse strand first, launch a fresh session, spawn the watcher, then attach.

`--no-attach` skips the terminal handover, which takes the CLI/Cobra Invariant's interactive-handoff exception the way `lyx loom start` does.
**Rationale:** Matches `lyx loom start`'s bootstrap shape; removing the corpse before relaunching avoids two strands under one name.

### D6 — Start prompt

**Decision:** A fresh launch's prompt is chosen in this order:
1. `--handoff <path>` if given.
2. The last completed handoff recorded in orch state (D10), if that file still exists.
3. The plain start stencil, which tells the session it is the hub orchestrator and to read the board.

In cases 1 and 2 the prompt is the resume stencil pointed at that file.
**Rationale:** Continuity across restarts comes free from the last cycle's handoff; lyx stays repo-agnostic by never naming `manifest/HANDOFF.md`, and the operator passes that file with `--handoff` when wanted.
The newest file in `.lyx/orch/handoffs/` can be a partial or aborted handoff that D11 left behind, so only a handoff whose phase completed counts.

### D7 — Context usage is read from the provider transcript, behind the Engine seam

**Decision:** The Claude engine reads context usage as `input_tokens + cache_creation_input_tokens + cache_read_input_tokens` of the last main-chain assistant message (sidechain entries excluded) in the session's current transcript.
It locates that transcript itself, from what the Stop hook records.
It follows the transcript through `/clear`, which starts a new session ID and transcript file.
A reading is taken from the transcript named by the turn end being evaluated, and only a turn end after the watcher's last injection is evaluated.
So the pre-clear handoff turn is never read after `/clear`, and the first post-cycle reading comes from the resume turn in the new transcript.
shuttleengine and orchengine see only a provider-invariant token count.
**Rationale:** Shuttle Provider-Seam Invariant; the Stop hook payload already carries the transcript location, so no guessing from directory listings is needed.

### D8 — Threshold is an absolute token count, configurable

**Decision:** `orch` config key for the threshold, default 150000 tokens.
Usage is checked on every turn end.
**Rationale:** The transcript does not state the model's window size, so a percentage has nothing reliable to divide by; 150k sits below Claude Code's auto-compaction point on a 200k window and leaves a 1M-window session plenty of room.

### D9 — Idle definition

**Decision:** The watcher may inject only when all of these hold:
- The newest event is a turn end, either `EventStop` or `EventWaiting`, never `EventAsk`.
- No newer event has arrived for at least the configured idle grace period, default 30s.
- A fresh pane capture, classified by a new engine probe, shows an idle session: the provider's input box present and empty, and no turn in progress.

Otherwise the watcher skips this poll and re-checks on the next one.
**Rationale:** The orchestrator nearly always has background watchers outstanding, so excluding `EventWaiting` would block cycling for hours.
The grace period keeps a cycle out of the operator's back-and-forth.
The probe prevents typing over an operator's unsent draft, into a permission prompt, or into a running turn.
The events file records no turn start, so after a turn end only the probe can see a new turn started by the operator or by a background task's completion.

### D10 — The cycle is a persisted state machine

**Decision:** Phases, persisted in the prime's `.lyx/orch/state.json` before each side effect: `idle` → `handoff-requested` → `clearing` → `resuming` → `idle`.
The persisted phase records the strand it belongs to, its entry time, and the events-file position at entry;
"a turn end after the injection" below means one past that position.
Each non-idle phase is bounded by the configured handoff timeout, measured from its entry time.
1. `handoff-requested`: type the handoff stencil (`/scribe:handoff <path>` followed on the same line by focus text), with the path a new timestamped file under `.lyx/orch/handoffs/`.
   Wait for the file to exist **and** a turn end after the injection, then record the file in state as the last completed handoff.
   D11 governs the timeout.
2. `clearing`: type the engine's clear-session choreography, then wait for D9's idle-session probe to pass.
   Move to `resuming` when it passes or the timeout expires.
   The startup-ready classification is not used here, since a pre-clear pane shows it too.
3. `resuming`: type the resume stencil pointing at the last completed handoff.
   Return to `idle` at the first turn end after the injection, or when the timeout expires, recording an abort reason in that case.

A watcher restarted against the same strand resumes from the persisted phase:
- `handoff-requested` or `resuming`: a turn end after the injection, or a turn in progress on the pane, means the injection landed, so keep waiting;
  an idle pane with no such turn end means it did not, so re-type the phase's stencil.
- `clearing`: re-type the clear choreography, then wait as above.

A fresh launch (D5's dead-or-absent branch) never resumes a persisted phase: it resets the phase to `idle` and records the abandoned phase as the last abort reason.
**Rationale:** A crash mid-cycle must neither double-inject a handoff nor strand a cleared session with no resume prompt.
Once the handoff is complete, a repeated `/clear` loses nothing and a resume prompt in an uncleared session is harmless, so neither phase needs proof that `/clear` landed.
A persisted phase describes one session's cycle; replaying it into a new session would type `/clear` or a resume prompt into a context that never asked for one.

### D11 — Never clear without a written handoff

**Decision:** `/clear` is injected only after the handoff file exists, is non-empty, and the handoff turn has ended.
If that has not happened within the configured handoff timeout (default 10 minutes), or the session asks a question instead, the cycle aborts back to `idle` with no `/clear`.
The abort is recorded in state, and the next attempt waits for the next qualifying idle turn end.
**Rationale:** A `/clear` without a handoff is the one unrecoverable failure here, since the context is simply gone.

### D12 — Handoffs are untracked and kept

**Decision:** Cycle handoffs live under the prime's `.lyx/orch/handoffs/`, never under `_lyx` or a tracked path; all are kept.
**Rationale:** The prime is on `main`, and `update-plugins.sh` requires a clean tree, so a tracked handoff would block deploys.
The Durable-vs-Ephemeral State Invariant puts never-tracked files under `.lyx`.
They are a few small files a day.

### D13 — Provider grammar stays out of orchengine

**Decision:** The clear-session choreography is a new `Engine` method in claudeengine.
The handoff instruction and resume prompt are stencils, read at call time per the Stencil Ownership Invariant.
Each injected stencil renders to a single line, so it can go through shuttle's single-line send path.
**Rationale:** Shuttle Provider-Seam Invariant: `/clear` is key grammar, while which skill writes a handoff is prompt content.

### D14 — Verbs

**Decision:**
- `start [--no-attach] [--handoff <path>]` behaves as D5 and D6 describe.
- `status` emits a JSON envelope: strand guid and liveness, watcher liveness, last context tokens, threshold, cycle phase, cycle count, last handoff path, last abort reason.
- `cycle` writes a request the watcher picks up, and runs one cycle at the next qualifying idle point regardless of threshold.
- `stop` removes the orchestrator strand; the watcher then exits on its own, and the next `start` resets any persisted phase as D10 states.
- `watch` is the daemon itself; it is started by `start`, not by the operator.

Every command carries a non-empty `Short` and JSON errors through `internal/output`.
**Rationale:** `cycle` gives the operator a one-command fresh context and a live-test hook; `status` answers "is it cycling and why not".

### D15 — Config

**Decision:** An `orch` config file with an embedded template: model, effort, threshold tokens, idle grace, handoff timeout, and poll interval.
It is loaded through `configengine.LoadOrTemplate` (degrading).
**Rationale:** Same class as shuttle and reed config, where a missing file should not stop the orchestrator.

### D16 — Cross-session messaging needs no lyx work

**Decision:** The strand-hosted session is an ordinary local Claude Code session, so other sessions reach it through Claude Code's `ListAgents`/`SendMessage` as today.
lyx adds nothing.
**Rationale:** The capability already exists; the roadmap's reed mailbox is a separate Someday item.

## Constraints

- CLI/Cobra Invariant: `orchcli` exposes `Command()`, `RunCLI`, `RunCLIIn`; every command has a `Short`; `RunE` checks `clihelp.ShouldAbort` first; the terminal handover in `start` is the only interactive-handoff exception, and must be added to that invariant's list.
- Told-Geometry Invariant: `orchengine` is told absolute paths and never imports `lyxcwd`; add it to the bound-packages list. `orchcli` resolves.
- Shuttle Provider-Seam Invariant: nothing Claude-specific outside `claudeengine`; `orchengine` never imports `claudeengine`.
- Durable-vs-Ephemeral State Invariant and Lyxdirs Single-Declarer Invariant: orch state, lock, requests and handoffs under the prime's `.lyx/orch/`, the `.lyx` literal only via `lyxdirs`.
- Stencil Ownership Invariant: the three stencils are seeded through `stencilstore`, read at call time.
- Config Strictness Invariant: add `orchengine` to the degrading list.
- Live-Substrate Spawn Observability: the detached watcher spawn and every tmux-touching probe log per that rule; any retry loop caps attempt count.
- Test Tier Purity: untagged tests use fakes only; real tmux/Claude exercises are `integration`/`smoke` tagged.
- Sandbox Suite Coverage: the new module is exercised by the sandbox suite or excluded with a written reason.
- Agent execution: interactive tmux only, never `claude -p`.

## Auto-mode assumptions

- Module name `orch`, verbs as in D14 (the board left the name open).
- Default threshold 150000 tokens, idle grace 30s, handoff timeout 10 minutes.
- `EventWaiting` counts as idle for cycling.
- No adoption of the operator's existing terminal session; the migration is: `/scribe:handoff` in the old session, exit it, `lyx orch start --handoff <that file>`.
- No automatic relaunch on process death; `lyx orch start` relaunches from the last completed handoff.
- Handoffs are kept indefinitely.

## Open risks

- Whether background tasks started by the session survive `/clear`, and whether their completion notifications land in the fresh context, is unverified.
  The handoff focus text tells the session to list its live watchers, so the resumed session can check before re-arming them;
  the smoke test verifies survival and records the outcome in the module doc.
- A `SendMessage` delivered in the sub-second gap between the handoff turn's end and `/clear` is lost from context.
- Whether a `SendMessage` address for the session stays stable across `/clear` is unverified; verify in the smoke test and record the outcome in the module doc.
- The transcript's usage fields and the Stop-hook payload shape are Claude Code internals and can change without notice; the parser must degrade to "usage unknown", which never triggers a cycle, rather than fail the watcher.
- If the configured threshold sits above the model's auto-compaction point, Claude Code compacts first; the cycle still works later, only less usefully.
  Even the default leaves a narrow margin on a 200k window: operator turns before idle qualifies, and the handoff turn itself, both spend it, so a busy session can compact first with the same outcome.

## Acceptance criteria

- `lyx orch start` from the prime launches one `orch` strand and one watcher, then attaches; a second `start` launches nothing new; `start` from a task worktree or the weft prime refuses on the envelope.
- `lyx orch start` after the strand died removes the corpse and resumes from the last completed handoff; `--handoff` overrides it.
- `lyx orch status` reports every field in D14.
- `lyx orch cycle` produces one full cycle at the next idle point: a new handoff file, then `/clear`, then the resume prompt naming that file.
- Unit tests (fakes, untagged) for the cycle state machine cover:
  - below threshold → no injection
  - above threshold, idle → handoff injected
  - newest event `EventAsk` → no injection
  - input box not empty → no injection, retried next poll
  - newer event inside the idle grace → no injection
  - handoff file written but turn not ended → no `/clear`
  - handoff timeout → abort to `idle`, no `/clear`, abort recorded
  - handoff written and turn ended → `/clear`, then resume prompt
  - idle-session probe never passes after `/clear` → resume prompt after the timeout
  - resume turn never ends → `idle` after the timeout, abort recorded
  - watcher restarted in each non-idle phase → follows D10's restart rules, with no duplicate injection when the earlier one landed
  - fresh launch over a non-idle persisted phase → phase reset to `idle`, abandoned phase recorded, nothing injected but the start prompt
  - aborted handoff file → never chosen as D6's resume source
  - pre-clear turn end after `/clear` → not evaluated for usage, no second cycle
  - manual `cycle` request below threshold → cycle runs
  - strand gone → watcher exits and records it
- claudeengine unit tests over transcript fixtures:
  - the token sum is taken from the last main-chain assistant message
  - sidechain entries are ignored
  - a transcript switch after `/clear` is followed
  - a malformed or missing transcript yields "usage unknown"
- claudeengine unit tests over pane-capture fixtures for the idle-session probe: empty box, box holding a draft, permission prompt showing, turn in progress.
- A `smoke`-tagged test drives a real session through `lyx orch cycle` with a low threshold and asserts handoff file, cleared context and resume prompt.
  It starts a background task before the cycle and records in the module doc whether the task and its completion notification survive `/clear`.
- Seam and invariant tests still pass: provider-seam, Told-Geometry, CLI help-tree, stencil registry.
- `docs/overview.md` lists the module; `CONSTRAINTS.md` lists `orchengine` under Told-Geometry and the degrading Config Strictness set, and `lyx orch start` under the interactive-handoff exception; `manifest/roadmap.md` gains the shipped item.

## Notes for the plan writer

- The go driver's detached spawn and `lyx loom start`'s bootstrap (`internal/loomcli/start.go`, `driverlaunch.go`) are the closest existing shape for D4 and D5.
- Batten's pre-run prime check (`internal/battencli`) is the pattern for D2.
- `shuttleengine.Runner` already has `Send`, `Inject` and the `sendVerified` path; `ModelSwitchSequence` is the precedent for a slash-command choreography on the `Engine` seam.
- claudeengine's `settings.go` writes the Stop hook; check what the hook records before deciding how the engine locates the transcript.
