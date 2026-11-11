```yaml
lens: fix-consistency
outcome: reported
```

Checking that the fix agrees with every sibling decision, criterion and cross-reference it touches (fix-consistency lens).

- **MEDIUM**: D5's await bullets, as rewritten. The fix made it possible for an ask to be `timeout` before a restarted row's chair runs the await, but the await's ready list was not updated to match.
  - Evidence: D7's new coverage entry sets a still-open ask to `timeout` before its deadline, and then fails the gate at cap with no accepted review, so the row restarts.
  - The restarted row's chair then runs `lym loom parent-review await`. D5 answers `ready: true` only for "`done`, `no-parent` or `not-sent`", or for "`pending-post` or `open`" at or past the deadline.
  - For an early `timeout` it falls to "Otherwise it waits… then answers `ready: false`", and the chair repeats this without end.
  - The new bullet "Its `ready: true` answer carries the ask's state" assumes every end state reaches `ready: true`. The first bullet's list needs `timeout` added.
- **LOW**: D4 criterion "an answer to an ask still `open` at or past its deadline sets the ask `timeout` and is refused, naming `lym loom decision add`" was not updated. Its twin criterion now says each refusal names `lym loom decision add` "and the run's resolved parent orch", as the rewritten D4 refusal bullet requires for every other refusal.
- **LOW**: the `## Auto-mode assumptions` line and the support log's Question ledger. The fix added new picks with no ledger entry: report ids `P1…`, the `parent_ids` finding key, a coverage gate entry that never passes on its cap, and closing the ask inside the gate before its deadline. Yet "the support log's ledger gives each one's rationale" for D4, D5 and D7. The Round 2 log lines record the fixes but not the picks' rationale.
- **LOW**: D7's "a finding key the review format gains" has no twin in the acceptance criteria or the plan-writer notes. The review format today allows only the finding keys `id`, `severity`, `class`, `location`, `summary`, `fix` and `origin`, and the `Finding` struct in `internal/burlerengine/verdict.go` has no `parent_ids` field. No Review criterion pins the format stencil gaining the key, so a plan could leave the chair unable to learn it.
- **LOW**: D4's copy bullet, "The copy joins the outputs the review row archives on a failed exit", and its criterion, "a failed review exit archives that copy". The copy now runs only once the review is accepted, and the failed exits in the review row (`failureExit` in `internal/shedadapters/burlerreview.go`) come before acceptance. So the archive clause and its criterion no longer name a reachable case, and the earlier "copy's exit set" rejection in the review rested on the table-end trigger.

I made no further tool calls.
