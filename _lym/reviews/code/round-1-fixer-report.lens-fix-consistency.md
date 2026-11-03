```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1–F3 change set for twins and sibling text left out of step with it.

no findings

The grep for the old wording ("never joined to", "spell it worktree-relative", "worktree-relative self glyph") matched only the spec line and the doc comment above `rootRelativeGlyphRemedy`.

- **Spec line 64 and the doc comment (`internal/planparser/validate.go:1356-1359`):** both still match the new message. The remedy is a worktree-relative self glyph, or the plain path `root:` joins, and the joined path is satisfied by disk, `Create`, `Rename` or part path.
- **Test marker `spell it as "`:** it matches the new message and no old text. The new `PathShapedRefGetsNoRemedy` row relies on the `ParseRef` guard returning `""` for a ref like `sub/dir`.
- **Unrun point:** I did not run that guard case. The gate was green with the row in place.
- **Spec's `//`-escaped path wording:** the spec says nothing about a `//`-escaped path. That fits the review's fix for F2, which only asked for the "present on disk or created by a card" change.

Tool calls (1 of 6):
1. `grep` for the old wording across the repo: my context held the edited files but not the rest of the repo, so I needed it to look for twins.
