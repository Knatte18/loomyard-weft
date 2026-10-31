---
round: 1
focus:
  - "internal/dobbyengine/starts.go: does a held task spawn no pass, clear no state wrongly, and start on the first tick after the record is gone with no backoff left (D1)? Check the hold path against the refused-start path for state and backoff handling."
  - "internal/dobbyengine/starts.go: is the hold notice posted once per text on the first tick, not resent on later ticks, and posted again after a changed board entry (D2)? Check where the noticed text is recorded and when it is reset."
  - "internal/battencli/dobbyreads.go and arm.go: does DobbyTasks fill the refusal only for a `done` status and leave it empty for no record, an undecodable status file, or any other state? Does the hold text equal the done-slug refusal a hand batten run prints, with its way forward, and is that text shared rather than duplicated (arm.go changes)?"
  - "internal/dobbyengine/view.go and internal/battencli/overviewdobby.go: do the rendered lines match D3 exactly (`waits on a start batten refuses: <text>`, `ready to start, last refused start: <text>`), and is `ready to start` never shown without a reason for a refused task? Check the expired-backoff condition."
  - "Tests (starts_test.go, dobbyreading_test.go, view_test.go, overviewdobby_test.go): does each acceptance criterion D1-D3 have a row that reaches it, including the no-backoff restart and changed-entry renotice? Flag duplicates of existing rows only by naming the covering test."
  - "Commit message and `_lym/landing/summary.md`: does the change description state the cause of the late notice and the origin of the other machine's run record (D4), and does every claim in it hold against the diff? Is the doc.go change accurate and within repository comment conventions?"
---

Seed from the decision record and diff stat; one commit in range touching `internal/battencli` and `internal/dobbyengine`.
