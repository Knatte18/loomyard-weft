```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each behavior change, I list the readers, callers and docs it forces.

**Per change, the set found**

1. `checkLivenessTick` now returns `OutcomeDied` for an untracked strand on a stop-marked handle.
   - **Callers:** `Wait` (`wait.go:388`) hands it to `finalizeAndRemoveDiedWindow` (`wait.go:410`, `wait.go:1851`). `awaitStartup` (`wait.go:672`) also calls it, but no `Stop` can reach a handle before `Start` issues it, so the new branch is dead there. Covered.
   - **`stopMarked` readers:** `finalize` (`wait.go:1796`) maps the Died to a `stopped` record through `stopOutcome`. Covered.
   - **Tests:** the existing mechanism-failure rows in `wait_test.go:318` and `startup_test.go:954` use unmarked handles and are unaffected. The new test in `stop_test.go` covers the marked path. Covered.
   - **Tripwire:** `completionsignal_enforcement_test.go` is updated to count three `OutcomeDied` returns, and the new return sits behind `allOutputFilesExist`. Covered.
2. Doc claims that `died` never covers an untracked strand were not all updated. Not covered; see the findings below.
3. PATTERN files:
   - **`pattern/PATTERN-shuttle-provider-seam.md`:** the Completion Signal section names the negative answers without per-branch counts, so it stays correct.
   - **Declared PATTERN changes:** the diff touches no `PATTERN.md` or `pattern/` file, and the directive declares none.
   - **Departures:** the commit message has no `Pattern-Departure:` trailer, and none is needed.

**Findings**

1. **LOW, `consistency`: `wait.go:5-6`.** The file header still says `died` is reserved for a strand reed still tracks, and that an untracked strand is "a mechanism failure, not a classification" with no exception. The diff added the exception to the `Wait` and `checkLivenessTick` doc comments and the package doc, but not to this header. Evidence: `wait.go:5-6` against `wait.go:226-227` and `wait.go:1241`.
2. **LOW, `consistency`: `engine.go:23`.** The `OutcomeDied` doc reads "a strand reed STILL TRACKS has a pane that is not alive…". I only saw the first line in the grep. The new untracked-and-stopped path now also yields `OutcomeDied`, so the claim is stale unless the line carries an exception. Evidence: `internal/shuttleengine/engine.go:23`.
3. **LOW, `consistency`: `docs/overview.md:420`.** The shuttle paragraph says reed's strand table losing the strand entirely is "never as `died`". That is now false for a handle whose own `Stop` removed the strand. This is a repository doc that describes the changed behavior, and the diff did not update it. Evidence: `docs/overview.md:420`.

**Tool calls**
1. A grep of `*.go` and `*.md` for "STILL TRACKS", `checkLivenessTick [OutcomeDied]`, `errStrandNotTracked` and `finalizeAndRemoveDiedWindow`. My context lacked the other readers, asserters and docs of the changed behavior.
