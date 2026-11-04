```yaml
lens: fix-code-correctness
outcome: reported
```

Task: run the fix-code-correctness lens over the fixer's change set, working from the context I already hold.

**Findings**

1. **LOW** — `TestEnsure_UnparseableDriftYamlFailsTheWalkNamingTheFile`, F7's Rewrite-error row.
   - The test asserts only that `Ensure` fails with an error containing `drift.yaml`, the tree is clean, no marker is tracked and nothing was mailed.
   - `reconcileHubWide` runs `ReconcileHubWideAt` before `applyReinterpretations`.
   - A drift file that does not parse most likely fails in the reconcile step, never reaching `change.Rewrite`.
   - If so, the test passes whether or not the `Rewrite` error is wrapped as a `FileError`, so it does not pin the `reinterpret.go:53-56` branch the finding named.
   - The gate showed only that the test passes. Nothing I hold shows which call produced the error.
   - Asserting `"reinterpretation"` in the error would show whether the branch is reached.

2. **LOW** — `readLogTail` in `steprun.go`, F3's tail read.
   - No test writes a log over `outputTailReadBytes` (64 KiB), which is the changed behaviour of the finding's first half.
   - The F3 rows use logs of a few hundred bytes, so they pass with the old `os.ReadFile` too.
   - Only the cut-from-start half of F3 is pinned by a test that fails with the defect in place.
   - Two small input-dependent behaviours of `readLogTail`:
     - When the window starts exactly on a line boundary, the first whole line is dropped.
     - When the window holds no newline, it keeps a partial line that may begin mid-rune, and the `[]rune` conversion turns the stray bytes into U+FFFD.
   - Both only affect the quoted tail, so I rate them no higher than LOW.

**Checked and found correct**

- **`boundOneLineKeepingEnd`**
  - The result is `…` plus the last `maxRunes-1` runes, `maxRunes` in all.
  - The over-long row wants `…` plus 399 zeros, which matches.
  - The short-last-line row computes 500 zeros / 500 zeros / `cause`, joined, and the last 399 runes are 391 zeros plus `" / cause"` (8). The expected `outputTailMaxRunes-1-len(" / cause")` is 391, so it matches.
  - Both rows fail if the cut goes back to the end.
- **`commandReport` callers** (`RunCapture`, `RunAcceptedCapture`, `RunCompare`)
  - They use the same signature and only gain the new tail behaviour.
  - An empty log gives `offset` 0, empty data and "printed nothing".
  - A missing log gives the "could not be read" clause. The new unreadable-log subtest provokes this by having the shell unlink its own log, and it fails if that clause is dropped.
- **`dirbase.go`**
  - `dir: null` and `dir: ~` carry tag `!!null` and are now skipped.
  - A quoted `"null"` is `!!str` and still treated as a literal directory.
  - Without the fix the `NullDirsAreKept` row would flag both keys, and `assertLineKeys` would fail on the unexpected flagged lines.
- **`reconcileNotifierFor`**
  - It behaves as the inlined closure did.
  - The other-hub row returns before `drivableLocation`, and it would post a mail if the board-dir check were removed, so the test catches that.
- **Missing-dir `OnEnd` guard**
  - `failCapture` returns before `OnEnd`, so the guard holds, and it would fail if `OnEnd` were called.
- **`TestRootPreRun_InstallsTheHubNotifiers`**
  - It fails if either install call is removed from the pre-run span.
- **No-notifier row in the reinterpretation test**
  - It relies on every earlier test restoring a nil notifier.
  - It would panic on a nil dereference if the nil check in `notifyReinterpretations` were removed.
- **Not-posted and notifier-error rows**
  - They only prove the walk still succeeds, because those paths only log.
  - They would catch a regression that made the walk fail, which is the realistic one.
- **F1 child-log assertion**
  - The envelope is written inside `OnEnd` after the record is written.
  - The `waitFor` on the child log covers that ordering.
  - Both the child's error text and `rec.Reason` come from the same `FailedCaptureError(rec)`, so the prefix match holds.
  - The `len(logs) != 1` check assumes `StartCapture` makes one child log per call. I did not read the spawn retry path in `spawn.go`, so a retry could make this flaky, though I have no evidence of one.

**Tool calls:** none. My context already held every file the lens needed.
