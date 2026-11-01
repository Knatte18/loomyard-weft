```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1 change set (`crucible/review-prompt-template.md`, `crucible/README.md`, commit 8ce1f333) for stale twins and sibling disagreement, as the fix-consistency lens.

no findings

- All five edit sites agree with each other. Lines 122, 129, 132 and 203 of the template and README line 201 now name `tiergate.Require(t, N)`. Line 148 of the template and `crucible/orchestrator-prompt.md` lines 119 and 124 already use `LYM_TEST_TIER`. The template names the grep with the full `tiergate.Require(t, 3)` form, which matches the review's `fix` text.
- Twin check: a grep of `crucible/` for `go:build tier`, `-tags tier`, `tier2 tag` and `build tag` finds nothing left. A grep for a bare `tier2` or `tier3` outside `LYM_TEST_TIER=` and `tiergate.Require` also finds nothing.
- Hits outside `crucible/` were not in the review's scope and are unrelated to the changed text. `internal/impactset` and `internal/planparser` handle `//go:build tierN` as a legacy or gate-only form.

Tool calls made (3 of 6):
1. `grep -rnE` for build-tag and `tierN` wording across the repo, to find twins outside `crucible/`. My context held only the review's named lines.
2. `git show --stat HEAD`, a grep for `tiergate.Require` usage, and a grep of `LYM_TEST_TIER` in `crucible/orchestrator-prompt.md`. They confirm the commit touched only the two files and that the sibling smoke commands use the run-time form. My context lacked the commit's file list and the orchestrator prompt.
3. `grep` of `crucible/` for a bare `tier2` or `tier3`, to check nothing else in the directory still uses the old wording.
