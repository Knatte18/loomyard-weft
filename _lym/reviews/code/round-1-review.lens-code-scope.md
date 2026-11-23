```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each change in the diff to the decision that asks for it, and each decision to the changes that build it.

**Decisions** (the board entry's `## Goal` points, plus the record's one `Added after Discussion` entry):
- G1: the stop path of `lym loom goto` may read an unwired reed runner.
- G2: `ClearReviewSession` may keep the PR-Review parent ask.
- G3: some dobby halt notices lack their autonomy stamp.
- G4: the goto hook and the halt-notice stamps have no tests.
- A1 (parent, added after Discussion): "No bug fixed". All four findings are already fixed on the base. This run adds one row to `TestGotoSpec_AdmittedHookClearsThePRReviewRecordOnlyForThatRow` for a recorded strand on a receiver with no runner, and changes nothing else. Where A1 differs from G1–G4, A1 wins.

**Per change:**
- **C1: the new row.** This is the `withStrand` fixture, the `PRReviewWithARecordedStrandStopsItThroughARunnerOfItsOwn` row, the `wantErr` branch and the doc-comment lines that describe it (`internal/loomcli/gotohook_integration_test.go:19-22, 30, 46, 55-61`). It traces to A1, which narrows G1 to pinning the case.
- **C2: a `held` field on every row.** The struct and table rework at `gotohook_integration_test.go:32-50` is forced by C1, because the new row seeds a different record than the other rows.
- **C3: whole-record comparison.** The `reflect.DeepEqual` check and the `reflect` import (`gotohook_integration_test.go:64`) are forced by C1. The new row's claim that the record stays whole includes `StrandGUID`, `StrandName`, `SessionID` and `Engine`, and the old three-field check could not see those. The table shares one assertion loop, so the existing rows' check also gets stricter. Nothing is loosened.
- **C4: hook-error check moves from `Fatalf` to `Errorf`.** At `gotohook_integration_test.go:55-58`, this is forced by C1's `wantErr` restructure. The existing rows still fail on an unexpected error. The only difference is that the loop goes on and reads the record, so no assertion gets weaker.

**Per decision:**
- G1: built by C1, as A1 narrows it.
- G2, G3 and G4: A1 records each as fixed and pinned on the base, so no change is owed. The summary names the pinning tests, and the files that hold them exist (the handler's grep found them).
- A1: built by C1. C2 to C4 are forced by C1. No change goes past A1's "one row, nothing else".

**Findings:** none.

**Tool calls:** none. My inherited context already held the diff, the decision record, the commit message and the grep that found the summary's named tests.
