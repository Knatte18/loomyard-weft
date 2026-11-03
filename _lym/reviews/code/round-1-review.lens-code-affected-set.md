```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens for round 1. For each changed signature, output, key or message, I list what reads or asserts it and whether the diff covers it.

**Per change, the set found**

1. **`CaptureRecord.WroteNoFile` (new key `wrote_no_file`).**
   - Writers: `RunCapture` (`capture.go:133`) is the only one. Every other `CaptureRecord` writer builds a fresh record or rewrites a fixture.
   - Readers: `emptyUnderSkips`, which `await.go:78` and `baseline.go:222` call.
   - The capture record is read by `ReadCapture` in `await`, `drive`, `Waive`, `dropCaptureSkipping` and `spawn.go:120`. None of them reads the new field, and none breaks on it.
   - A record written before this change has no key and reads back as false. It takes the old plain-capture Stuck, so there is no wrong rescope and no lost field. `internal/state` keeps unknown keys on rewrite.
   - Docs: `doc.go` lists the field at line 22 and describes it at line 34. Covered.
   - Tests: the three files that assert it were updated. Covered.
   - Specs, stencils and docs: a search of `contracts`, `plugins` and `docs` finds no description of the capture record's keys. Nothing is left stale.

2. **The failure reason of a capture that wrote no file.**
   - The old text `exit 0 but wrote no file to <dir>` is the prefix of the new one, so `HasPrefix` checks keep passing.
   - Every other string in the repo naming it is in the touched tests. Nothing else asserts or parses it.
   - The way forward keys on the flag and not on the text, as the design says. Covered.

3. **`awaitWaysForward(onBranchPoint, rescope)`.**
   - Its only caller is the `stuck` closure at `await.go:80`, and its only test path is `TestDriftAwait`. Covered.
   - Existing rows keep the old wording; the plain-capture and defer-only strings are unchanged.
   - `deferWay` is a new constant for text that was a local variable. Nothing else duplicated it.

4. **The foreground Stuck in `captureAgain`.**
   - It reaches `rescopeWaysForward(true)`. This is sound, since `RunCapture`'s `Check` ran first and HEAD was the branch point.
   - `Defer` (`verbs.go:190`) admits a Drift-Baseline halt through `CaptureHaltPrefix`, and the reason keeps that prefix.
   - `RecordScope` (`verbs.go:67`) admits a blocked Drift-Baseline or Drift-Await, so every verb the new text names is reachable at both halts.
   - `--scope full` or a shorter `--skip` list drops the stale capture record through `dropCaptureSkipping`, so the follow-up `lym drift capture` and `lym loom resume` start clean.
   - `--scope off` skips the drop. Drift-Await and Drift-Baseline both return Done on `ScopeOff`, so the stale record is harmless.

5. **The capture's skip list.**
   - Every caller of `RunCapture` that sets `request.Skip` passes `scope.Skip`: `driftcli/cli.go:408` and `loomcli/driftwiring.go:155`. The record's `Skip` and the count in the reason therefore match the scope record.
   - Callers using `SkipList(scope)` (`cli.go:243`, `cli.go:297`, `driftwiring.go:190`) serve compare and side capture, which never write `WroteNoFile`. Not affected.

6. **Rule documents.**
   - `PATTERN-refusal-way-forward` binds `internal/shed*`, `internal/webster*` and `internal/loom*`, not `internal/driftengine`. The new Stuck reasons carry `way forward:` clauses anyway.
   - Neither new way forward uses a denied form from the closed list.
   - `PATTERN-test-speed`: the new rows sit in existing tier tables and add no test function. See finding 1 for the one uncovered behavior.
   - `PATTERN-doc-go-documentation` is not triggered, since the `doc.go` lines describe behavior and do not set a rule for other code.
   - The branch carries no `Pattern-Departure:` trailer and no `PATTERN.md` or `pattern/` change, and the review prompt declares no PATTERN changes. Nothing is missing.
   - The commit message claims bear out against the diff:
     - the flag, the reason and the Stuck are all in the diff;
     - the "while HEAD is the branch point" condition is carried by `rescopeWaysForward`.

**Findings**

1. **LOW, `consistency`.** The single-skip wording branch in `noFileReason` (`capture.go:146-149`, "1 test") has no row that would fail if it broke.
   - `capture_test.go` exercises only a two-test skip list.
   - `baseline_test.go:409` hand-writes a "skipped 1 test" reason into a fixture, so it asserts nothing about `noFileReason`.
   - This belongs to the tests lens. I raise it only because the affected set is the one test file that asserts the message.

2. **NIT, `consistency`.** The `doc.go` line at `doc.go:105` ("swaps those ways for the scope changes the Capture section names") is a cross-reference to another section of the same file.
   - It does not follow the repository's self-contained comment rule that the rubric's comment-convention bullet points to.
   - The Drift-Await section names no wording of its own for the scope changes.

I found no other member of the affected set left wrong.

**Tool calls (1 of 6)**
1. `grep` over `contracts`, `plugins` and `docs` for the drift capture ways forward and the capture record's keys. My context lacked those trees, since the manifest covered only `internal/`.
