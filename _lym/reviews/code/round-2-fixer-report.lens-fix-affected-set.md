```yaml
lens: fix-affected-set
outcome: reported
```

No findings. The affected-set check found no other site that the fix had to cover.

Task: rerun the set searches behind the set-premised findings, F1 (every site claiming a Sketch keeps its dates) and F3 (every site claiming the draft fallback is untested).

- **F1:** A repo-wide grep for "dates stay", "issues, dates" and similar wording, excluding the review files, found no remaining Sketch "dates stay" claim. The `doc.go` and `summary.md` claims were the only two, and both are corrected. A second grep for Sketch within 200 characters of dates, in either order, over `*.go`, `*.md` and `*.yaml` also returned nothing. The other date hits are unrelated: rename keeping dates, the age columns, and `cli.go:203` listing dates among the fields an upsert accepts.
- **F3:** The "covered only by reading" and "fallback to draft" greps found no other site claiming that coverage is missing. The `summary.md` bullet is the only one, and it is deleted.
- **Unchecked:** F2 is a single missing test row, not a set premise, so I did not search it.

Tool calls:
1. Bash grep for "dates stay", "issues, dates" and "fallback to draft" claims across `*.go` and `*.md`: my context lacked the repo-wide sweep for other sites.
2. Bash grep for Sketch near dates across `*.go`, `*.md` and `*.yaml`: my context lacked a check of docs, skills and the overview for a second wording.
