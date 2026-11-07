```yaml
lens: code-affected-set
outcome: reported
```

**Task:** code-affected-set lens. For each change, I looked for the callers, tests, docs and rule documents it forces an edit in.

**Per change, the set found and whether it is covered**
- **`onlyKeepLinesAfter` removed from `guards.go`.** No caller is left. `onlyDirectiveLinesAfter` serves guard, seam and watch alike. Covered.
- **`targetsFor` gained a `watched` parameter.** The only callers are `DeriveTargets` (`targets.go:93`) and `targets_test.go:171`. Both are updated. Covered.
- **`holdsTierTest(…, skipped)`.** The only caller is `tierrun.go:382`. `emptytier_test.go` is updated. Covered.
- **`watchPhase`, `runRequest` and `cancelOnPassedLease` gained a period argument.**
  - Every production caller in `detachfixer.go` and `detachrun.go` passes `seams.leaseRead()`.
  - `cardfixer_test.go` defines its own `fixerRig.runRequest`, not the changed method, so nothing there breaks.
  - Covered.
- **`attachLineWaiting` signature and wait text.** The only callers are `attachLine` and the integration test. `termwindow_test.go:70` still asserts `waiting up to 30s` and passes: `30*time.Second` prints as `30s`, and `sleep 1` is unchanged. Covered.
- **`guardScanRefusal` `GuardMisplaced` message reworded** (it no longer says `//lym:guard marker at`).
  - No test asserts the old text. `test_integration_test.go` asserts the `tier2` message and the new watch message.
  - Covered.
- **New `//lym:watch` marker.**
  - Landed: the `PATTERN.md` entry, the `pattern/PATTERN-test-speed.md` section, `gatecli/doc.go` and `impactset/doc.go`.
  - Related docs naming the guard markers are `loomshed/verifygate.go:72`, `loom-plan-spec.md:335`, `webster-body-implementer.md:71` and `cmd/testtiming/main.go:87`. None of them is falsified.
  - `docs/overview.md:401` and `:878` mention the guard step but not watch. They do not contradict the change, and I raise no finding.
- **Commit trailers.** No `Pattern-Departure:` trailers are present or needed.

**Findings**

1. **LOW, consistency, `pattern/PATTERN-test-speed.md:117-118` (the "Repository-scanning tests carry `//lym:guard`" section).**
   - It says a narrowed gate's guard step "runs every marked test by name … whatever the impacted set".
   - The diff changes this for a test carrying both markers: `scanGuards` (`scope.go`) and `targetsFor` (`targets.go:1216`) drop a watched guard test that the changes leave alone.
   - `TestCSharpFixtures_CompileWithDotnet` carries both markers, so the pattern text is now wrong for it.
   - Fix: add one sentence there, or in the watch section, saying a watch marker on a guard test lets the guard step drop it too. `gatecli/doc.go` already says this.
   - Evidence: `pattern/PATTERN-test-speed.md:117`, `internal/gatecli/doc.go` ("The guard step leaves out a watched guard test"), `internal/impactset/targets.go:1216`.

2. **LOW, consistency, `internal/gitexec/gitexec.go:57-62`.**
   - `armDeadline` is a package-level `var` replaced by tests.
   - `pattern/PATTERN-test-speed.md` "Time is injectable" says a production interval, timeout or clock a test would wait out is "never a package-level `var`, so tests keep `t.Parallel`".
   - `SetDeadlineForTest` documents that it is not parallel-safe.
   - `startStallTimer` and the stall seams already do the same, and `TestRun_RemoteBounds` is serial. The new seam is therefore consistent with the neighbouring code but not with the written rule.
   - Fix: either note the `gitexec` exception in the pattern, or leave it as precedent. A reviewer should decide.
   - Evidence: `gitexec.go:57`, `export_test.go` doc of `SetDeadlineForTest`, `PATTERN-test-speed.md` "Time is injectable".

**Tool calls**
1. A grep for the callers of every changed signature and for tests asserting the old `guardScanRefusal` message. My context lacked the callers outside the diff.
2. A grep of `.go` files that mention `lym:guard` or `lym:seam`, and of the termwindow tests for the wait text. My context lacked the non-diff files.
3. A grep of the `//lym:` marker mentions in docs, stencils and specs. I needed to check whether any of them is falsified by the change.
