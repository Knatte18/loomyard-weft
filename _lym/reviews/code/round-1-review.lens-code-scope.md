```yaml
lens: code-scope
outcome: reported
```

Task: trace each hunk of the diff to the decision record's Goal points (code-scope lens).

**Per change**

- C1, `internal/boardengine/render.go:49`, swaps `fsx.AtomicWrite` for `os.WriteFile`. It traces to Goal 2: "fix it where the cause is; change how lym writes board files only if the cause is there." The summary places the cause in the temp-and-rename write (`summary.md:11-26`) and rules out settings and a symlink (`summary.md:28-33`).
- C2, the `RenderToDisk` doc comment (`render.go:37-39`) plus commit b05e3b1d, which re-breaks it. Forced by C1: the comment now describes the changed write.
- C3, the `internal/boardengine/doc.go` paragraph (diff lines 9-10). Forced by C1: the package doc already describes what every write renders, and the paragraph adds how it writes.
- C4, `TestRenderToDiskRewritesAnExistingFileInPlace` (`render_test.go`). Forced by C1: it is the test of the new behaviour.

**Per decision**

- Goal 1 (find the cause and state the evidence in the report). Not in the diff; the report lives at `_lym/landing/summary.md:7-33`, which covers all three candidates the Goal names. Built.
- Operator decision (find the cause first, without assuming one). The summary checks the operator's expected cause, the settings, first and rules it out with evidence (`summary.md:30-33`). Built.
- Goal 2 (fix where the cause is). C1 builds it, for rendered board files only. Partial, see F1.
- Goal 3 (how the operator confirms by hand). `summary.md:62-68`. Built.
- Added after Discussion entries: none in the record.

**Findings**

- F1. LOW · `scope` · `internal/boardengine/render.go:49`
  - **Summary:** The fix covers only the files `RenderToDisk` writes. Decision records (`decision-<slug>.md`) still go through temp-and-rename, though they sit in the same `_bolt` root the workspace opens in the markdown preview. The same cause therefore still leaves an open preview of a Sketch stale, which falls short of Goal 2's "fix it where the cause is".
  - **Evidence:**
    - `internal/boardengine/decision.go:62` and `internal/boardengine/recordupsert.go:66` keep `fsx.AtomicWrite`.
    - `_lym/landing/summary.md:40-41` keeps their atomic writes on purpose, because lym parses them back.
    - `summary.md:60` lists the stale-preview risk itself.
    - Board files link these records (`render.go:834-836`, `sketchLink`).
  - **Why only LOW:** The summary's reason holds: lym parses those files back, so it needs atomic writes. The entry asks about "the board file the operator is looking at", and rendered files are its main subject. The fix text should do one of two things: write decision records in place under the board write lock, or leave the code and have the parent record the exclusion with `lym loom decision add`.

Tool calls: none.
