```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

- **F6 (stale wording):** a repo-wide search for "captured at", "profile captured" and "no accepted profile" finds no remaining member in code, stencils or docs. The only hits are two lines in `docs/research/drift-whole-review-2026-11-04.md`. That file records a point-in-time review, which the rubric's measurement-report exception leaves alone, and the review's F6 did not name it.
- **F1 (stem match):** the match rule lives only in `testFiles`, so the fix covers it everywhere. Its callers are `requireBaselineTest`, `keepAcceptedOutput`, `dropAcceptedOutput`, `buildAcceptedProfile` (twice) and `RunAcceptanceCapture` at `capture.go:289`. The stricter rule reaches all of them, and no other code in the package matches a stem with `path.Ext`.
- **F3 (drop callers):** my context already holds the earlier grep. `dropAcceptedOutput` had two callers, `keepAcceptedOutput` and `Withdraw`, and both now pass the new `except` argument.
- **F2, F4, F5:** their premises are single sites, not sets. I left them unchecked.

Calls made:
1. Repo-wide grep for stale capture wording. I made it because the review named only five sites and my context held no search outside the drift packages.
2. Grep of `path.Ext` and `testFiles(` in drift code. I made it because my context lacked the full list of `testFiles` callers.
