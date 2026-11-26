```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of the fix change set.

1. **MEDIUM** — `internal/gatefixer/rowgate.go`, the doc comment of `waitOnly`. It still says "`markOnPass` says whether a passing or pending inner marks the live record terminal; a failing one always does." The fix made that false.
   - `waitThenRun` now skips `markTerminal` when `outcome.deadReason == DeadReasonInterrupted && !replace`, and the non-carrying entries reach it with `replace == false`.
   - The new `WrapRowGate` text ("unless the fixer ended interrupted") and the new test `AnotherEntryFailingOverAnInterruptedFixerLeavesItLive` both contradict "always does".
   - The fixer rewrote the `afterFixer` sentence directly below and left this one.

2. **LOW** — `internal/shedadapters/bouncer.go`, `judgeCall`, the attach branch left beside the changed `runErr` branch.
   - The new doc says "A run error without such a judgment fails the call".
   - The `summary.md` line added by the fix says "A judge run that returned an error with no verdict fails the call the same way".
   - `attachErr` from `b.cfg.Shuttle.Attach(spec)` still goes to `b.degrade(ctx, "shedadapters: bouncer judge attach probe failed", …)`, which returns a counted `Stuck`.
   - I did not read whether `Attach` returns a waited live judge's run error through that value. If it does, an attached judge whose wait errors still spends bounce budget, and both sentences overstate the behaviour.

3. **LOW** — `internal/websterengine/verifyfixer.go`, the `FixVerifyAwait` doc comment. It cites `DeadReasonInterrupted` alongside the package's own constants.
   - `websterengine/runplan.go:57-59` re-exports only `DeadReasonAsking`, `DeadReasonTimeout` and `DeadReasonDied`.
   - `internal/runplan/digest.go:24-26` likewise lists only those three.
   - The only `DeadReasonInterrupted` is `gatefixer`'s, which the new webster test has to import. So the doc names an identifier this package does not have.

Everything else I checked agrees with the change: the `gatefixer` package doc, `Await`'s order, the `shedadapters` `doc.go`, and the CLI help for `fix-verify` and `fix-plan`.

Outside my lens: the bouncer's seed pass still turns a seed run error into `Stuck` (`bouncer.go:1906`). The review did not ask about it and the fix did not change it.

Tool calls (3):
- grep for statements that a judge run error degrades: my context held only `bouncer.go` and its docs in slices.
- grep for statements of the orphan sweep rule: my context lacked the other files that describe the sweep.
- grep for dead-reason lists and fixer-wait wording: my context lacked `runplan`'s constants and any other documents that list them.
