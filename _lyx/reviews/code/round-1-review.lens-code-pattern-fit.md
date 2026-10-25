```yaml
lens: code-pattern-fit
outcome: reported
```

Lens: code-pattern-fit. Rules checked, held or broken:

- **Declared change `PATTERN-cwd-resolution`: held.** `PATTERN.md` carries the declared text verbatim, with the `(test)` marker and the background link. The overview line measures 357 bytes, under the 360 cap (`MaxEntryLineChars`, `internal/pattern/check.go:26`; the count is in bytes, which is at least the character count). The background file `pattern/PATTERN-cwd-resolution.md` gains the own-marker, `NamedWorktree`, write/git-read and import-set lines.
- **`PATTERN-leaf-packages`: held.** `lyxcwd` now imports `weftname` (`internal/lyxcwd/anchor.go`). Its import cap is updated in `leaf_enforcement_test.go`, and the background file lists the import.
- **`PATTERN-fabric-vocabulary`: held.** The `weftname` importer rule is updated in `pattern/PATTERN-fabric-vocabulary.md` to cover `lyxcwd`.
- **`PATTERN-lyx-temp-root`: held.** No added non-test line uses `os.TempDir`.
- **`PATTERN-cwd-resolution`, no raw cwd calls: held.** I found no added non-test `os.Getwd`, `rev-parse --show-toplevel` or `lyxcwd.Location{` literal. I only grepped the added lines; I did not run the card-2 scan test.
- **`PATTERN-cwd-resolution`, engines never import lyxcwd: held for what I could check.** The diffs of `dobbyengine`, `hubgeom` and `pairteardown` add no `lyxcwd`, `fabricengine` or `weftname` import.
- **Commit messages and departures: held.** No `Pattern-Departure:` trailer appears in the range, so no departure needs judging.
- **CLAUDE.md "Docs land in the same commit": held.** `doc.go` changes land for `fabricengine`, `fabriccli`, `beamcli`, `adoptcli` and `dobbyengine`, plus `docs/overview.md` and `docs/shared-libs/lyxcwd.md`.
- **`PATTERN-cwd-resolution` and `PATTERN-fabric-git`, marker write and weft git only in `fabricengine`: broken, one finding below.**

Findings:

1. **LOW, class `consistency`:** `internal/hubforge/reanchor.go:51-58`.
   - `ReanchorPair` writes `.lyx-anchor` and commits it in the weft with raw `gitkit.MustRun(... "git", "mv"/"add"/"commit")`, from outside `fabricengine`.
   - The declared text says "only `fabricengine` writes or git-reads it", and `PATTERN-fabric-git` limits git on weft to `fabricengine`.
   - The plan's Shared Decisions licenses this fixture ("Fixtures for a second anchor"), and `hubforge` is a test-fixture package.
   - It holds only if the PATTERN scans exempt fixture packages like `hubforge`. I did not check the scans, and neither the declared text nor the background file names such an exemption.
   - Fix: record the fixture exemption in `pattern/PATTERN-cwd-resolution.md`, or confirm a scan already covers it.

Tool calls (2):
1. Counted the overview line length, grepped the added non-test lines for banned calls, listed new imports in `dobbyengine`, `hubgeom` and `pairteardown`, and listed `Pattern-Departure` trailers; the manifest held none of these.
2. Read `internal/hubforge/reanchor.go` and listed the production files that use `AnchorFileName`, to see who writes the marker outside `fabricengine`; the manifest did not cover `hubforge`.
