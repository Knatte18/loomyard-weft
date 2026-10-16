# Fix the eight creel stage 1 re-test defects

Each defect from the 2026-10-16 creel stage 1 re-test now meets the expectation its board entry states.

## What changes

- **Journal tail (#677):** the repair at journal open cuts a torn last line whether or not it ends in a newline.
  The fold's skip rule and the repair's cut rule share one definition of the last non-blank line and one parse test, so a line the fold skips is the line the repair cuts, including valid JSON that is no record, such as `[1]`.
  A malformed line anywhere else still fails the start with its way forward.
- **Unwritable stdout (#681):** a creel refusal whose JSON envelope cannot be written to stdout writes `creel: <error>` to stderr and exits 1.
  This holds for every verb; `read` leaves the mail unread.
- **Stale stall (#682):** a receiver's standing stall records when a probe last read it.
  A new `inject --wait`, an `expired` record and its answer name the stall only when that reading is no older than twice the probe backoff ceiling; a stale stall is dropped.
  A waiter is told each change it sees once, and also a stall that persists from before it arrived.
- **Connect wait (#683):** the spawn seam returns an exit signal beside the pid, and `creelcli` reaps the detached daemon in the background, logging its exit with pid and status at Info.
  The client waits on the backoff timer or that signal.
  When the daemon exits other than by a lost start race (exit 0), the client makes one last connect and then returns the existing daemon-log error without waiting out the backoff.
  A lost start race keeps today's backoff, since the winner may not answer yet, and a seam with no signal keeps it too.
- **Carried records in `log` (#684):** a carried record adds a `carried` entry to the event list, both on fold and when retention carries it in a running daemon, so `log --id` finds it.
- **Removed-holder sweep (#685):** after a first miss the next sweep runs after a 10 s confirm interval, and the backoff ceiling drops from 4 minutes to 50 s.
  New mail never lengthens the wait to the next sweep.
  A removed holder is settled within 60 s while listings succeed.
  The two-misses-in-a-row rule and "a failed listing counts toward no streak" are kept.
- **Inject reasons and TTL (#686):** `shuttleengine.IdleProbe` reports a pane that shows no input box, and the creel adapter maps it to a new stall `no-input-box`, after too-short and draft in precedence.
  The stall only names a reason; it changes no idle answer.
  A live receiver's wait before its next probe is capped at the remaining TTL of its soonest queued inject, never below the one-second floor, so an inject expires at its TTL.
- **llm smoke test (#678):** the smoke helper adds its strand with `--anchor own-window` (the #673 workaround), the idle wait prints the last pane capture when it gives up, and before it the test accepts a startup gate through a new `shuttleengine` seam.

Docs, `PATTERN-daemon-wakeups.md` (creel worker and sweep lines) and the C4, C6, C8, C10 and C11 lines of `SANDBOX-CREEL-SUITE.md` are updated with the code.

Closes #677
Closes #678
Closes #681
Closes #682
Closes #683
Closes #684
Closes #685
Closes #686

## Design decisions to review

- The gate seam in `shuttleengine` runs the provider's own startup-gate classification and accept sequence on a strand the caller names.
  For claude that covers the folder-trust gate and the bypass-permissions gate.
  Claude shows the bypass gate only to a strand whose flags or settings already ask for it, so the seam grants no permission the strand did not request.
- The seam is exported but its only caller is the llm smoke helper.
  A seam-enforcement test in `internal/shuttleengine` pins that no production package outside `shuttleengine` calls it; `creelcli` reaches it through `shuttleengines`, never `claudeengine`.
- Accepting the folder-trust gate adds the strand's working directory to the operator's Claude trust list, because llmkit does not isolate Claude's config.
  For the smoke test that is the temp hub's path.
- The 10 s confirm interval shortens the two-miss guard from at least 30 s to at least 10 s.
  A strand absent from reed's persisted state for longer than that and re-added under the same name loses its held mail back along its path.
- The `no-input-box` stall also fires while a claude pane boots or a dialog is up; each change journals one `stalled` record.
- A start that fails before it tries the lock ends the client's wait even if another client's daemon is starting; that verb returns the daemon-log error unless its one last connect reaches that daemon.
- Dropping a stale stall rather than marking it as past, and naming a reason for #686 rather than only documenting the outcome, were chosen where the board allowed either.

## To check by hand

- Run `lyx gate test --tags llm ./internal/creelcli`.
  No gate runs the `llm` tier, so #678 is compiled by `go vet -tags llm` but not proven; if the printed pane capture shows a third cause, that run names it.
- Two flaky tests, a sweep row and a scheduler inject row, failed in a run and are not caused by this change.
  Confirm they are the known flakes if they fail again.
- `reed-add-parentless-placement` also edits `SANDBOX-CREEL-SUITE.md` and may touch reed-add calls in test helpers; whichever lands second merges the other's lines.
  Once #673 is fixed, the `--anchor own-window` in the smoke helper goes with the other workarounds.
- Until #680 is fixed, a sandbox re-check from inside a strand pane runs the production `lyx`, so check a dev build's daemon from the shell.
