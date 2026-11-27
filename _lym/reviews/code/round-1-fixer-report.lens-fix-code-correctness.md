```yaml
lens: fix-code-correctness
outcome: reported
```

Task: fix-code-correctness check of the F1–F5 test changes.

no findings

I traced each changed path against the defect it guards:
- **F1:** with the arguments swapped, the query asks `head-1..older`, gets false and ends `pushed`, so the new row fails.
- **F2:** a swallowed ancestry error leaves `err` nil. `errors.Is` is then false and short-circuits before `err.Error()`, so the subtest fails without panicking.
- **F3:** both omit flags reach the intended `sessionRef` branches. Without the trim, the empty-engine row would see a doubled space and fail.
- **F4:** the added fragment fails if the halt goes back to naming only the run directory.
- **F5:** each row's setup logs no `feat` line other than the one from the call under test, so dropping that call's trace line fails the row.

I made no tool calls. This check is by reading the code in context; I did not mutate it to watch the tests fail.
