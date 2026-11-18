```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. Trace each change in the diff to the goal points of the board entry.

**Decisions.** The answer key is the decision record alone. Its `## Goal` holds three points, and the record has no `Added after Discussion` entries.
- G1: find the cause and state the evidence in the report.
- G2: fix it where the cause is, and change how lym writes board files only if the cause is there.
- G3: say in the report how the operator confirms the fix by hand.

**Per change:**
- C1, `internal/boardengine/render.go:49`: `fsx.AtomicWrite` replaced by an in-place `os.WriteFile`. Traces to G2.
- C2, `render.go:37-39` (the `RenderToDisk` doc comment) and `internal/boardengine/doc.go:341-342` (the package doc paragraph): forced by C1, since they document the changed write.
  Commit `b05e3b1d` only re-breaks those comment lines.
- C3, `internal/boardengine/render_test.go:45-76` (`TestRenderToDiskRewritesAnExistingFileInPlace`): forced by C1, as the test of the new behaviour.

No change traces to nothing. No `Pattern-Departure:` trailer is present or needed.

**Per decision:**
- G1: no code change. It is carried by the report: `_lym/landing/summary.md` "Cause and evidence" and "Ruled out", plus the body of commit `28c270f4`. Satisfied outside the diff, as the goal asks.
- G2: built by C1, which C2 and C3 follow. It is built only partly; see F1.
- G3: no code change. It is carried by `_lym/landing/summary.md` "How to confirm by hand". Satisfied.

**Findings**

F1. Severity LOW, class `scope`.
Location: `internal/boardengine/decision.go:62` and `internal/boardengine/recordupsert.go:66`.
Summary: G2 asks for the fix where the cause is. The diff removes temp-and-rename only for the rendered files. Decision record files, `decision-<slug>.md` in the same board directory, are still written by temp-and-rename through `fsx.AtomicWrite`. Every discussed entry's board file links its record, and the workspace opens that link in the markdown preview, so its preview can go stale after a board write that rewrites it.
Evidence: the `fsx.AtomicWrite(b.boardPath, name, string(record))` call at `decision.go:62`, and the same call at `recordupsert.go:66`. The summary concedes the gap under "Open risks": "Decision record files still go through temp-and-rename, so a preview of one can go stale after an edit".
Fix: either rewrite decision record files in place too, or record a parent decision that the gap is out of scope.

Tool calls: none. My inherited context already held the diff, the record, the board entry, the summary and the `fsx.AtomicWrite` call sites.
