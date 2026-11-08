```yaml
lens: fix-consistency
outcome: reported
```

Task: the fix-consistency lens over the F1–F3 change set.

**LOW** — `internal/beamcli/land.go:36`, the `DraftDescription` seam comment, was left unchanged.
- It still reads "writes a starting point over the beam's change description". After the fix, the draft refuses when `summary.md` already stands, so "over" now misstates the seam.
- This is the twin of the `describe.go` comment that was edited ("where no description stands yet").
- Evidence: the grep hit `internal/beamcli/land.go:36`, and the edit list in context has no change to that file.

**LOW** — `contracts/orchrole/role-beams.md` was not checked.
- The review says it tells the orch to run `--draft` "first". If it describes `--draft` as rewriting or refreshing the draft, it now disagrees.
- My grep for overwrite or replace wording near "draft" found no hit in any `.md` file, so the likely outcome is no conflict. I did not read the file.

Other twins agree with the change:
- `cli.go` has the help text, the refusal bullet and the flag text.
- `doc.go` has the `beamcli` text.
- `describe.go` has the comment.
- `BeamDraftDeps` has the `Slug` field, which `beamDraftDeps` fills.
- The test rows pass `Slug: "my-beam"`, and the integration test calls the `DraftDescription` seam through `hubLandDeps`, not `BeamDraftDeps` directly.
- The `landingshed` `doc.go` line 618 only says "told seams", so it is not stale.

Tool calls:
- 1 grep across `.md` and `.go` files for draft overwrite or replace wording, to find twins the edited files didn't show.
