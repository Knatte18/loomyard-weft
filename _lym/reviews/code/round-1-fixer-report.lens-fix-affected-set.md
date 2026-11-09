```yaml
lens: fix-affected-set
outcome: reported
```

No findings on the code sets (F2, F3, F4). The fix left four sites stale, and they are the same set as F8 and F9:

- **LOW**, `internal/loomrecipe/doc.go:55`: "Raddle-File … files the approved proposal on the board and ends the run's entry". This is the same unqualified claim F9 fixed in the recipe comment, but the arras family filing does not end the entry.
- **LOW**, `internal/boardengine/raddlefile.go:2`: the file header says "and ends the run's entry". The header was not touched by the fix. The lines below it say a family filing turns the entry into the project, so the file now disagrees with itself.
- **LOW**, `internal/loomshed/raddlefile.go:139`: the `Call` doc says "step 6 ends the run's entry, leaving the row Done". It then adds "A family filing skips step 6", so this one is already qualified and needs no change.
- **LOW**, `internal/fabricengine/replay.go:250` and `internal/fabricengine/doc.go:1761`: both say Raddle-File "ends the run's own entry" and reason from an entry showing done at origin.
  - The fix did not touch them.
  - A family-filed entry is a project with no status, never done, so this reconcile reasoning may not cover it.
  - This could be out of scope for the review, but it is the same claim as F8. The fixer should either qualify the text or say why it is left.

Other findings checked:

- **F2**: `stageAndFile` has one caller, `Call`, and the status is cleared right after `repairFamily`, so every `ClaimStatus` path is covered.
- **F3**: `endsAsFamilyProject` has one caller, `checkRunLocks`. `endsAsSketch` was out of scope per the review.
- **F8 and F9**: `docs/overview.md:1302`, `raddle-recipe.yaml:4` and `wiring.go:232` are fixed and consistent with `loomcli/doc.go:66`.

Unchecked: F1, F5 and F6 (test coverage rows, not set-shaped) and F7 (disputed, no code change).

Tool calls:
1. A grep across the repo for "ends the run's/own entry" and "entry done" sites. My context held only the files the fixer read, not the whole set of sites that make this claim.
2. A grep for the callers of `ClaimStatus`, `stageAndFile`, `endsAsSketch` and `endsAsFamilyProject`. My context had no search across the repo for callers.
