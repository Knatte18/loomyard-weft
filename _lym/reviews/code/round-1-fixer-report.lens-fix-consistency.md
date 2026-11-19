```yaml
lens: fix-consistency
outcome: reported
```

Fork task: check the fix-round change set for stale siblings and twins left unchanged, through the fix-consistency lens.

**Findings**

1. **LOW**: `internal/landingshed/deps.go:243`, the `RecordLanding` doc comment.
   - It says "emptyDiff reports that the landing changed no path against the parent head merged into the task worktree."
   - F1 changed the computation to code paths only, through `ChangedCodePaths`. `finalize.go` and `doc.go` were updated to say "no code path", but this comment was not.
   - It sits two lines above the new `ChangedCodePaths` doc in the same file. Its wording was not changed in the F1 commit `58a43111`.

2. **LOW**: three sibling descriptions of the same flag still say "no path", and the flag now means no code path.
   - `internal/shedrun/landed.go:24`, the `LandedCommit.EmptyDiff` doc.
   - `internal/shedrun/doc.go:48`, "marks a landing that changed no path on the parent branch".
   - `internal/dobbyengine/passes.go:77`, the `EmptyLanding` doc.
   - Each reads as if records paths count. A reader of these comments would assume the pre-fix semantics.
   - `landnotice.go:308` and `landnotice.go:318` carry the same wording, and the dobby line says "no file of its parent branch changed".
   - Also stale: the Warn log text at `finalize.go:764`, "the landing changed no path on the parent branch".

**No finding**

- Seam-registry lists: grep for `Landing.ChangedPaths` found only `internal/loomcli/wiring_seams_test.go` and `internal/battencli/wire_seams_test.go`. Both received a `Landing.ChangedCodePaths` twin.
- `internal/testkit/envkit/envkit.go`'s `nilLegal` does not list `Landing.ChangedPaths`, so it needs no twin for the new seam.
- `RecordLandedCommit` is the only production caller of the new write path. `WriteLandedCommit` still has other callers, and its "replacing" doc is accurate.
- `internal/darnengine/template.yaml` already describes `report_dir` without a records-directory claim. The F5 refusal is documented in the `LoadConfig` doc, which was updated. The template line does not conflict with it.

**Tool calls**
1. Grep for "changed no path" wording across the Go, md and yaml files, plus the `report_dir` lines of `template.yaml`. My context lacked the sibling comments in `dobbyengine`, `shedrun` and `deps.go` that restate the flag, and the template comment.
2. Grep for `WriteLandedCommit` and `RecordLandedCommit` callers, plus `git show --stat 58a43111`. My context lacked the full caller set and the F1 commit's file list.
