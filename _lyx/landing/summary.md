# creel stage 2: orch and batten notices go through creel

Every notice and every line lyx types into the orch session now goes through creel.
The orch watcher keeps only its cycle logic: tracking tokens, choosing a `refresh`, `distill` or automatic cycle, waiting for the note, sequencing the reload, its state and its lifecycle.
Its notice queue (`orchengine.QueueNotice`, `QueueConditionalNotice`, the `notices/` directory) and its idle probe before typing are removed.

## What changes

- **Creel types into orch strands.**
  Reminders, injects and the new command injects reach an orch strand; stage 1's withholding and the `withheld` record go.
  A stage 1 journal holding `withheld` records still folds, and the fold acts on none.
  `lyx creel inbox` loses its withheld flag.
- **The reminder carries no content.**
  It reads `[creel] N unread: #id from <sender>, … — run lyx creel read`, at most five entries and `+K more`.
  Under a hold the command reads `lyx creel read --from <owner>`.
  `lyx creel log` keeps showing subjects.
- **Command injects.**
  A closed set of provider session commands (`/clear`, `/compact`, `/reload-plugins` and the like) that a program sender queues for an orch strand, typed through shuttle's session operations.
  A hold or command inject on an orch strand is refused from any sender but a program.
  `lyx creel log` shows command injects.
- **A Go-client-only `withdraw` verb.**
  A program removes its own queued injects and its own unread mail to one receiver.
  It answers the id and subject of each message removed.
- **A typed refusal code** on creel's response (`internal/creelengine/protocol.go`), set only when a strand claim comes from a process in no strand pane.
  The Go client returns it as an exported error that `NotifyPrime` matches with `errors.Is`.
- **Shuttle events carry a hook-side time.**
  `shuttleengine.Event` gains a time field that only `Runner.ReadEvents` sets, from the engine's `ParseSessionSignals`; `shuttlefake` gains a matching `ParseSessionSignals`.
  The watcher's reload steps compare it with creel's `sending` time to confirm a step.
- **The watcher's cycle runs over creel.**
  The note request is mail answered by post (`written <path>` or `defer`).
  The `/clear`, `/compact`, plugins, color, skills and pointer steps are command or text injects, and a creel hold keeps other deliveries off the orch during a cycle.
  The pause-resume line is creel mail.
- **Every notice caller moves to creel mail:** batten's notices, `orchcli.NotifyPrime` and its callers (burler, webster, loom driver hand-back, loom landing, shuttle's hold and dialog notices, hibernate).
  Tender's orch revive is a checked text inject; `lyx tender revive`'s report for the orch drops its conditional-delivery and watcher-live fields and reports the inject id.
- **The hub notice log goes** (`.lyx/notices.log`, `shedrun.NoticeLog`, battenshed's `NoticeLog` seam).
  The orch role's fallback reads unread creel mail, and `lyx creel log` for notices already read.
- **Orch CLI and state.**
  `lyx orch start` runs two withdraw rounds before launching and fails, launching no session, when the first one fails; the watcher retries a failed second round.
  The orch state loses `stuck_by_hold` and `stuck_by_send` and records a failed second fresh-launch withdraw or a failed release.
  `lyx orch status`'s `stuck` names the stall reason creel last reported for the pending inject, or the creel failure holding the cycle.
- **Docs and PATTERN.**
  The orch stencils and the package docs (`creelengine`, `creelcli`, `orchengine`, `orchcli`, `battenshed`, `tenderengine`, `shuttleengine`) describe the new paths.
  `PATTERN-durable-vs-ephemeral-state` gains the operator's sentence that nothing a run depends on lives only under `.lyx`, and `PATTERN-guarded-interrupt` is adjusted.

## Design decisions to review

- The hub-pause handling stays in the watcher: it keeps the `Session.SessionIdle` readiness reading and the guarded interrupt, a bounded departure from "the idle probe goes".
- The watcher always signs as its own program, `ORCHWATCH`, whatever `$LYX_STRAND_NAME` it inherited.
- While any live strand's pane process is unknown or any worktree's reed state is unreadable, creel refuses every sender outside a pane, the watcher included; a cycle stalls and the orch state records the reason.
- The note request costs the orch two hops, a reminder turn and a read, where the old typed instruction cost one turn; a hard cycle at the context cap spends one more turn of context before the note.
- Notice files queued under `.lyx/orch/notices/` at the deploy are not delivered.
- The shortened operator wording in the `PATTERN-durable-vs-ephemeral-state` sentence departs from the operator's text; restore it if preferred.

Out of scope: notice wording, de-duplication of drivers' stop reports, agent-to-agent messaging, `lyx shuttle send`/`interrupt` and direct typing into non-orch strands (stage 3, `creel-agents`).

## To check by hand

- `withdraw` against a receiver the sender never wrote to: the decision record leaves the behaviour for an unlisted receiver unstated.
- A fresh launch that leaves stuck text in the orch pane: nothing clears it before the first inject.
- The record of an inject's outcome against its `sending` record is ambiguous for an inject read but not yet marked sending; a `withdraw` can race it.
- The creel seam mappings in `orchcli`, the read-acknowledgement failure path in the watcher, and a withdrawn resume line across a retried fresh launch were each fixed late (`orchcli` seam, `orchengine`); review those fixes and their tests.
- Whether shuttle's verified session operations report a landed `/clear`, `/compact` or `/reload-plugins`, which start no turn, as landed rather than `ErrNotLanded`.
- The llm tier compiles but was not run for the cycle change; run the tmux and llm smoke tiers (for example `TestSmokeCreelClaudeStrand`) by hand.
