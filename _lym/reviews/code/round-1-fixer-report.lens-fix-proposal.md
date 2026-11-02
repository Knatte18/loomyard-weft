```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals for F1–F5, which the fixer carried out as proposed.

One finding:

- **LOW** — `internal/shuttlecli/hookbatching_test.go`, `TestHookBatchingVerb`: the F1 test runs `hookBatchingCmd()` on its own, so the `PersistentPreRunE` skip in `internal/shuttlecli/cli.go` is still unpinned.
  - The review's Issue text for F1 named that skip, which lets the verb run outside a hub, as one of the unpinned behaviours.
  - The proposal as written ("runs `hookBatchingCmd` through cobra") leaves it out, and the fixer followed the proposal.
  - Evidence: the skip is `cmd.Name() == shuttleengine.BatchingHookVerb` in `cli.go`'s `PersistentPreRunE`. A verb rename or a dropped clause would send the hook down hub resolution and an error envelope on stdout, and no test would fail.
  - A fix would add a row that runs `Command()` with `hook-batching` args outside a git repository.

I tried to build a failing case for the other proposals and found none:

- **F2:**
  - The split-line row `{text, ~, Glob}` fails with grouping by message id removed. The text line would become its own no-call message, the run would be 1, and the row would turn silent.
  - The sidechain and no-id "not counted" rows fail with the skip removed. The skipped lines would count as lookups, giving a run of 3, so the test's silent expectation would fail.
- **F3:** The proposal said to parametrise the budget or expose the function through `export_test.go`. `readTranscriptTail` already takes `budget`, and the test file is `package claudeengine`, so direct testing is sound.
  - Budget 6 over `aaa\nbbb\nccc\n` leaves `b\nccc\n`, so the first line is dropped and `ccc\n` remains.
  - Budget 2 over `aaa\nbbb` leaves `bb`, which has no newline and returns nil.
- **F4:**
  - `unterminated.go` has 19 newlines plus an unterminated last line, which makes 20. Without the unterminated-line count it would be 19 and fire, so the row pins it.
  - `exact.go` stops the loop at `lines == limit`, so it is not short.
  - The order row `Read, Grep, Read` sliced wants the lookup nudge. A swapped order would give the short-file slice text, so the row catches the swap.
- **F5:** I found nothing to break in the rows or the bounds table.
  - The proposal said to turn the `TestPrepare_BatchingNudgeHook` refusal rows into a bounds table. The fixer instead added `TestValidateBatchingNudge` and kept the Prepare row, which pins the separate "refused before any artifact" behaviour.

Tool calls made: none. My context already held every changed file and the review.
