```yaml
lens: fix-affected-set
outcome: reported
```

No findings in the `start.go` or `tidy_test.go` sets. One finding, on a site the review did not name:

- **LOW**, `internal/hubgeom/dobbygeom_test.go:60`: the `TestDobbyGeometry` doc comment still says "the lock, state and marker paths lie under the named worktree's `.lym/beam/` directory".
  - This is the same stale wording F2 fixed in `dobbygeom.go`, and the geometry has no marker path.
  - F2's own search, for "marker" next to the geometry paths, finds this twin outside the review's named location. The fix should read "lock, state and log paths and the tidy directory".
  - Evidence: `grep -rniE "marker (dir|path)|..."` over `internal`, `cmd`, `docs` and `PATTERN.md` returned this line.

Unchecked: nothing.

The fix covers every other member of F1's set. Of the remaining hits, `dobbygeom.go:65` is correct, because the marker paths belong to `PawlPaths`, and `internal/dobbyengine/doc.go` has no stale directory wording. F3's set is a single assertion, and the `board-tidy-` prefix no longer appears in `internal`, `cmd`, `docs` or `pattern`.

Tool calls:
1. `grep` for stale marker-directory wording across `internal`, `cmd`, `docs` and `PATTERN.md`: my context held only the three named sites.
2. `grep` for "marker" in the `dobbyengine`, `dobbycli` and `beamcli` packages: to confirm no other stale wording sits beside the dobby paths.
