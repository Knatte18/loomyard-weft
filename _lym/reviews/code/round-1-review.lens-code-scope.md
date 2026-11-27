```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens, which maps each change in the diff to the board entry's goal points.

**Per change:**
- **C1** (`internal/shedadapters/multillm.go:210` and `:225-227`): the reason now reads "timed out and was stopped", and `timeoutWayForward` now reads "starts the row over in a fresh session", with its comment. It traces to goal point 1: make the timeout reason say the session was stopped and that `lym loom resume` starts the row over.
- **C2** (`internal/shedadapters/multillm_test.go:360,367,376`): the three timeout rows now pin the new wording. It traces to goal point 3, checking the tests that pin the current wording, and C1 forces it.
- **C3** (`internal/shedadapters/doc.go:34-36`): the package doc's outcome-mapping sentences for a timed-out chair. C1 forces it, because the old line described the reason C1 changes.

**Per decision** (decision record alone, from the board entry's `## What to do`; the record has no `Added after Discussion` entries):
- **G1**, make the reason say the session was stopped and resume starts the row over: built by C1 and C3.
- **G2**, the alternative of leaving a busy timed-out chair live for resume: not built. It is an either-or with G1 ("Alternatively"), so G1 settles it and there is no gap. The change description gives the reason in `_lym/landing/summary.md`, under Design decisions.
- **G3**, check the tests in `multillm_test.go` that pin the current wording: built by C2. A repository-wide grep found no other test or caller that asserts the old wording.
- **G4**, consider whether a row's wait on its own background work should count against the chair's limit: no code change. The item only asks for a consideration, and `_lym/landing/summary.md` (Design decisions) records one with the limit left unchanged, so there is no gap.

**Findings:** none. Every change traces to G1 or G3 or is forced by C1, and the record asks for nothing that the diff leaves unbuilt.

**Tool calls:** none; the inherited context held the diff, the decision record and the change description.
