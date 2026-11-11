```yaml
lens: fix-consistency
outcome: reported
```

no findings

Checks:
- `refuseUncontainedPath` appears nowhere in `.go` or `.md` files after F4.
- `hubRel` is declared once, in `hubworkspace.go`.
- The `.cmd` and symlink rows in `retiredlaunchers_test.go` are consistent with `retiredLauncherFileNames` and the sweep's `Lstat` logic.

Tool calls (1):
- One `grep` for the deleted name and the `hubRel` declarations, because my context shows neither the rest of the repo nor the post-move state of `hubRel`.
