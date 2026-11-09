```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

I reran the two set-premise searches and found nothing the fix missed.

- **F1 (reasons that leave a test unrun):** `stopReason` is the only source of `reasonBudgetSpent` and `reasonCancelled`, and `retries()` at `redundancyreport.go:169` now checks both. The other unrun reasons are `reasonNoSlot` and `reasonNotRun`. Both come with an error line (`redundancy.go:199`, `redundancy.go:541`), so `len(r.Errors) > 0` already retries them.
- **F3 (`secondUnit` sites):** a search for `testRequest{` finds two production constructions, `cli.go:102` and `rungate.go:28`, and both set `secondUnit: time.Second`. Every other hit is a `_test.go` file. The detached runner goes through `newCommand`, which reaches `cli.go:102`, so it has no separate site.
- **F3 residual:** the new scenario test pins the exit code and red flag through `runTest`. It does not pin the `secondUnit` literals. That is a limit of the fix, not a missed member of the set.

I did not search the F2, F4, F5, F6 or F8 sets (kinds of test rows, not call sites).

Tool calls:
1. `grep` for `testRequest{` across the repo and for `stopReason`/`reason*` in `internal/gatecli`: to enumerate the `secondUnit` construction sites and the unrun-reason sources, which my context did not hold in one list.
