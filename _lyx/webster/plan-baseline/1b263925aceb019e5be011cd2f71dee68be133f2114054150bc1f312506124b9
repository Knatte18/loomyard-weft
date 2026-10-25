# Card 1 — lyxcwd reads each worktree's own marker

**Edit:**
- `internal/lyxcwd#readRecordedAnchor` -> `func readRecordedAnchor(hub, worktreeName string) (anchor string, found bool, err error)`
- `internal/lyxcwd#resolveCore`
- `internal/lyxcwd#AnchorFileName`
- `internal/lyxcwd#TestReadRecordedAnchor_MarkerSemantics`
- `internal/lyxcwd#TestLeafInvariant_AllowlistOnly`
- `internal/lyxcwd#weftnameImportOwners`
- `internal/lyxcwd_test#TestResolve_AnchorScenario`
- `PATTERN.md`
- `pattern/PATTERN-cwd-resolution.md`
- `pattern/PATTERN-fabric-vocabulary.md`
- `docs/shared-libs/lyxcwd.md`
- `docs/overview.md`
- `internal/fabricengine/doc.go`

**Create:**
- `plan:internal/lyxcwd#NamedWorktree` -> `func NamedWorktree(hubPath, name string) (*Location, error)`

**Uses:**
- `internal/lyxcwd#ValidateAnchorRel`
- `internal/lyxcwd#ErrStaleAnchorMarker`
- `internal/lyxcwd#buildLocation`

**Intent:**
D1, D2 and D3's constructor.
`readRecordedAnchor` takes the worktree's name and reads, in order: the worktree's own marker at `weftname.SiblingPath(hub, worktreeName)/.lyx-anchor`; else the legacy Bolt marker `<hub>/_board/.lyx-anchor`, with the `ErrStaleAnchorMarker` guard applying to this Bolt read only; else nothing found, so the caller answers `"."`.
An own marker that fails `ValidateAnchorRel` is an error naming the file, as the Bolt read's is; an own marker empty after trimming counts as absent and the read falls through to Bolt.
`resolveCore` passes `filepath.Base(workTreeRoot)`, so `Resolve` and `ResolveWorktree` read the resolved worktree's own marker.

`NamedWorktree(hubPath, name)` is the Location of the worktree `name` of the hub at `hubPath`: it reads that worktree's anchor through `readRecordedAnchor`, applies no cwd gate, spawns no git and needs neither the worktree nor its records sibling to exist (both absent answer the Bolt value or `"."`).
`RepoName` is derived from `hubPath` exactly as `buildLocation` derives it.
It is the one way production code outside `lyxcwd` gets another worktree's Location from here on (Shared Decisions).

`lyxcwd` production imports grow by `internal/weftname` only: `TestLeafInvariant_AllowlistOnly` admits it, its file comment's import-cap sentence names `internal/weftname` beside `internal/dotgit`, and `weftnameImportOwners` admits `internal/lyxcwd` in `TestEnforcement_FabricVocabulary`'s weftname import rule.
`lyxcwd` is outside the fabric-vocabulary owner set, so its new code, comments and docs name the records sibling without the bare side tokens (the `weftname` package name is exempt).

Tests: `TestReadRecordedAnchor_MarkerSemantics` drives `NamedWorktree` instead of the private reader, over a temp hub with no git, and gains rows: own marker wins over a Bolt marker; own marker absent falls back to Bolt; own marker empty falls back to Bolt; an invalid own marker is an error wrapping `ErrInvalidAnchor`; a stale Bolt `.fabric-anchor` beside a present own marker is no error, and the existing stale row still hard-errors with no own marker.
One row also asserts the returned Location's `HubPath`, `WorktreeName` and `RepoName`.
`TestResolve_AnchorScenario` gains a last step, after the stale-marker step, so the own markers it writes never mask the Bolt-only stale guard that step pins: a second linked worktree of the fixture repo gets its own marker `frontend` while the first's own marker names `backend`; each resolves its own `AnchorRel`, and `Resolve` from each one's anchored directory refuses the other's with `ErrCwdOutsideAnchor`.

Docs: `anchor.go`'s file comment and `AnchorFileName`'s doc say where the marker lives and the read order; `docs/shared-libs/lyxcwd.md` describes the read order and `NamedWorktree`; `docs/overview.md`'s resolver list names `NamedWorktree`; `pattern/PATTERN-fabric-vocabulary.md`'s weftname-import line and `internal/fabricengine/doc.go`'s weftname-import subset sentence add `internal/lyxcwd`.
`pattern/PATTERN-cwd-resolution.md` replaces "`lyxcwd` imports the standard library and `internal/dotgit` only" with the standard library, `internal/dotgit` and `internal/weftname`, says a worktree's anchor comes from its own weft-root marker with the Bolt fallback, that `NamedWorktree` is how another worktree's Location is had and that only `fabricengine` writes the marker or reads it from git.

`PATTERN.md`'s `PATTERN-cwd-resolution` line becomes:

```
- `PATTERN-cwd-resolution` — Resolving a cwd, worktree root or anchor: only `internal/lyxcwd`, from the worktree's own weft-root `.lyx-anchor`; only `fabricengine` writes or git-reads it; engines get absolute paths, never import lyxcwd; hub containers and workspace files stay hub-reachable only. (test) — [background](pattern/PATTERN-cwd-resolution.md)
```

**ImpactSummary:** one private reader re-signed with its one production caller and one test caller in the same package; no hub carries an own marker until card 6, so every existing resolution answers as before.
