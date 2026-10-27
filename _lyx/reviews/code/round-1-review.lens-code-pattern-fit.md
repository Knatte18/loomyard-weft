```yaml
lens: code-pattern-fit
outcome: reported
```

**Lens: code-pattern-fit.** The list below shows each rule I checked and whether it held or broke.

**Rules checked**
- `CLAUDE.md` PATTERN is authoritative: held. The `PATTERN.md` entries and `pattern/` files carry the renamed vocabulary (`lymcwd`, `lymdirs`, `LYM_*`, `lym gate test`, `PATTERN-lym-temp-root`). No `lyx` spelling is left in them, and the one `LyX` hit is the deploy test's editor guard.
- Declared PATTERN changes landed: held. The directive lists no add, change or remove beyond the rename, and the renamed entries match their code names. The `pattern/` background files changed text only.
- `PATTERN-durable-vs-ephemeral-state` (`_lym` tracked, `.lym` untracked, same subpath, only `internal/lymdirs` names either): held. `hubgeom/dobbygeom.go` builds `.lym/beam/` from `lymdirs.DotLymDirName` and `lymdirs.BeamDirName`. The dobby log and tidy paths hang off `beamDir`.
- `PATTERN-cwd-resolution` (anchor file and `lymcwd`) and `PATTERN-leaf-packages`: held. The package rename is consistent in the imports I read.
- CLAUDE.md "Docs land in the same commit": held for the dobby change.
  - The `doc.go` for `dobbyengine` now says the report is `.lym/beam/tidy/<UTC timestamp>.md`.
  - `internal/beamcli/doc.go` says `dobby.log` is in `.lym/beam/`.
  - `docs/overview.md` still says only "its `dobby.log`", which stays true.
- CLAUDE.md "Production: only `update-plugins.sh`" and "Build: cgo": held. The `update-plugins` scripts and `deploy-dev` only change the name.
- CLAUDE.md "Markdown: semantic line breaks": held for the changed `.md` files I sampled.
- Commit messages and departures: held. No commit carries a `Pattern-Departure:` trailer, so no unknown or non-forbidding trailer exists. The messages match their diffs.
  - The first commit's description of the dobby paths and the `go.sum`/`docs/research/` exclusion is accurate.
  - The reflow commit's "no word changed" claim holds. The sorted word counts of the removed and added lines in `a2ae85464` are identical.
- Scope against the board entry: held for this lens. I found no migration code, no `go.sum` or `docs/research/` change, and `TidyDirName` replaces `MarkerDirName` as D3 says.

**Findings**

1. LOW, consistency: `internal/dobbyengine/start.go:20` — the `LogFileName` doc says "the dobby log file in the marker directory". That directory is gone, and the file now sits directly in `.lym/beam/`. Fix: reword it to "the dobby log file in the worktree's `.lym/beam/` directory".
2. LOW, consistency: `internal/dobbyengine/start.go:242` — the `Live` doc says "A worktree whose marker directory does not exist yet". The code stats `filepath.Dir(geom.LockPath)`, the `.lym/beam/` directory. Fix: say "whose `.lym/beam/` directory".
3. LOW, consistency: `internal/dobbyengine/start.go:258` — the `SpawnDetached` doc says "appended to the marker directory's log". Fix: say "appended to `geom.LogPath`".
4. LOW, consistency: `internal/hubgeom/dobbygeom.go:21` — the `DobbyGeometry` doc says "The lock, state and marker paths lie under that worktree's `.lym/beam/` directory". The geometry now has `LogPath` and `TidyDir` and no marker path. Fix: say "The lock, state and log paths and the tidy directory lie under…".

These four are leftovers of the dobby change that the script's name rewrite could not catch. Each is a comment that now describes a removed directory.

**Tool calls (2 of 6)**
1. Word-multiset comparison of the removed and added lines of commit `a2ae85464`. My context had the commit message but not its diff.
2. `git grep` of `PATTERN.md` and `pattern/` for "marker", "dobby" and "LYM_". This checked the PATTERN text for stale dobby wording and `LYM_` consistency, which my context lacked.
