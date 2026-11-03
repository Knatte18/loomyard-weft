```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens on the diff (`spawn.go` `seed`/`isBeam`, plus the test).

**Per-function list**

- `seed`, prime slug, `primeResolved` true: regenerates the hub file, `launchPath` is the hub file, `windowDir` is its dir. Same as before. Correct.
- `seed`, `primeResolved` false (any slug, prime name unresolved): returns the bare folder, no regeneration, no error. Same as before for the prime. Correct. A beam whose prime cannot be resolved also degrades quietly, which matches the doc comment.
- `seed`, task slug (`found` false, or `found` with `Beam` false): `isBeam` returns false, so the bare folder. Correct. `isBeam` adds a `ReadOriginFor` call to every task seed, but it creates and writes nothing, so it only costs a read.
- `seed`, beam slug, `RegenerateHubWorkspace` succeeds:
  - `hubWorkspaceFolders` sets `pair.beam = origin.Beam` for every listed pair, and `layout.beams` holds each such pair. So the regeneration writes `BeamWorkspacePath(l, slug)` for any beam whose folder is present and resolved. Both are checked on the way in (the anchor dir exists and `ResolveWorktree` works), and `seed` has already written that worktree's config.
  - Launch path and `windowDir` are the beam file and its dir (the hub dir). Correct.
  - The `l.RepoName` that `BeamWorkspacePath` uses is the same one the regeneration uses, because both take the same `l`. Correct.
- `seed`, beam slug, regeneration fails (lock held, prime settings unreadable, write failure): returns `result` plus the error. `Spawn` returns the error and launches nothing. Matches the doc ("every error on either path is returned"). Correct.
- `seed`, beam slug whose pair is absent from the layout but has a marked record. Examples: its worktree is not a direct child of the hub, its anchor dir is missing, or `List` marks it prunable. The regeneration skips it, so the beam file is not written. `seed` still returns `BeamWorkspacePath` as the launch path. The launcher then opens a file that does not exist, or a stale one, and no error is raised. This is an unlikely edge, because `NamedWorktree` and `writeWorktreeConfig` have already succeeded for the slug. A stale file would also be removed only when the beam is gone, per the `removeStaleBeamWorkspaces` doc. `seed` has no check that the file exists after regeneration. Finding F-LOW below.
- `isBeam`, `ReadOriginFor` returns an error (for example the weft `.weft` lock directory is missing): logs a Warn and returns false, so the bare folder. Matches the doc. The Spawn doc comment says "to read a pair's origin record, is logged and degrades", which fits.
- `isBeam`, prime slug: never called, because `isPrime` short-circuits. Correct.
- `Spawn`: launches `result.launchPath`, then `placeWindow(l, windowDir)`. For a beam the window dir is the hub dir, the same as for the prime. Correct.
- Test `checkBeamOpensBeamWorkspace`, with `MarkBeam` on an `AddPair` pair: asserts Seed writes the beam file with the title `TST: <slug> workspace`, and launches nothing. It then removes the file, runs Spawn, and asserts one launch of the beam path, one window-command run, and the beam as first folder. The sequence is coherent. The title assert depends on `ReadShortname` returning `TST` in the hubforge hub. This is the same mechanism the prime test relies on, which I did not re-read. State left behind: a `beam-workspace` pair with the beam mark and a beam file stay in the hub for the later steps. They would now appear in the hub workspace's folder list unless it is a beam with no parent, which is listed in the hub file. The later `checkPrimeWritesHubWorkspace` may therefore see an extra folder. I could not confirm that without reading the test. Test coverage is for the other lens, so I record only the risk.

**Findings**

- F-LOW, design, `internal/ideengine/spawn.go:103-107`: `seed` derives the beam launch path from `BeamWorkspacePath` and never checks that the regeneration wrote it. A marked beam the layout skips (worktree outside the hub, missing anchor dir, prunable) therefore gets a launch of a nonexistent or stale file, with no error. Evidence: `hubworkspace.go:318-339` skips such pairs, and `hubworkspace.go:381-399` builds beam files only from listed pairs. Fix would be an existence check, with a fallback to the bare folder or an error.

No other correctness defects found.

**Tool calls**
1. Read `internal/ideengine/spawn.go` lines 66-115, because the manifest lacked the head of `seed` (`resolveSpawnTarget`'s results and `primeResolved`).
