```yaml
lens: code-correctness
outcome: reported
```

Task: lens code-correctness, running each added or changed branch with a concrete input. I found no defects.

**List (function or branch, case run, verdict):**
- `runCheck` start mark, `check.go:155`: the record has `State` running and no `Running`, so `checkMarkOf` takes the `len(parts)==0` case and gives `adopt check: starting` with `Start` = the record's `Start`. Correct.
- `runCheck` `Started` observer, `check.go:185`: tier 1 after tier 0 green. `tierStart` is set before `save()`, so the mark reads `adopt check: tier0 green 40s · tier1` with the tier's start. Correct.
- `Ended` observer, `check.go:190`: `Running` is cleared and the tier appended before `save()`. The mark takes the `default` case with the done tiers and no start, so tmux shows no seconds between tiers. Correct.
- Reconcile-failure path, `check.go:177`: `State` is failed, no tiers, `finish` marks. The text is `adopt check failed before any tier ended; lym beam status shows why`. Correct.
- Gate exit 0 with no tiers, because the green record carried them: the mark is `adopt check green` alone. Correct. If a carried tier is reported through `Ended` with a zero wall time, the bar would show `0s`, which is cosmetic.
- Gate exit non-zero with a tier started but not ended, so `Running` is cleared by `finish`: the mark shows the done tiers as failed, or the "before any tier ended" text if none. Correct.
- `finish` after a failed `writeCheck`, `check.go:171`: the mark is still set and the write error is returned. Correct, since the mark is display only and matches what the operator saw.
- `mark()` closure, `check.go:150`: it reads `record` and `tierStart` by reference, so it always sees current values. A `Mark` error is only logged, so the record and notices are unchanged. The test pins this.
- `checkMarkOf` text and state mapping against D2: the starting, in-flight, between-tiers, green and failed texts all match. Green drops the outcome words and failed keeps them. Correct.
- `SetCheckMark` with a non-zero start: it sets three options, with the start in epoch seconds. Correct.
- `SetCheckMark` with a zero start: it unsets the start, so no stale seconds remain. Correct.
- `SetCheckMark` with an empty text: it unsets all three options. Correct. Production never clears.
- `SetCheckMark` with no session: `requireSessionLocked` returns an error, nothing is written, and the caller logs it. Correct.
- `SetCheckMark` when the first call succeeds and a later one fails: the options can be left half-set. This is harmless and next mark overwrites it.
- `checkMarkFormat` / `checkMarkStyleFormat`, `bar.go`: the commas sit only inside nested `#{?}`, and the text is substituted after parsing, so a comma in the text cannot break it. An unset `@lym_check` gives an empty branch, so there is no mark. The tier2 test exercises the format on real tmux. Correct.
- `hubCheckDeps` / `primeReed`: a config or geometry error is deferred and returned from each `Mark` call, which is logged and ignored. The check still runs. Correct.
- `adoptGateTiers`: an unreadable config or a missing `adopt` key gives nil with a Warn. The tiers are left out and the start proceeds. Correct. `StartSuite` returns `suiteStart{}` on an `ensureCheck` error, so there is no message and `suite_warning` is kept.
- `ensureCheck` `AlreadyRunning`: set only on the found-running branch. Correct.
- `suiteMessage`: with tiers `[0 1 2]` the sentence matches D1 exactly. With none it gives `started in the prime;` and with `AlreadyRunning` it says `already running`. Correct.
- `cli.go` envelope: `message` is added only when `SuiteMessage != ""`, which `adopt` sets only on a successful start, so a failed start has no message. Correct.

**Findings:** none.

**Tool calls (1):**
1. Read `internal/adoptcli/check.go` lines 120–231: to see `runCheck` and the observers in full, which the manifest diff only showed in part.
