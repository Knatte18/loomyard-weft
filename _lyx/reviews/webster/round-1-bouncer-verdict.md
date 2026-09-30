---
verdict: APPROVED
rationale: "All eight round-1 findings (F1-F8) are fixed in commits ff4b81084..dd61add11 and hold on HEAD, the plan's full verify block (build, vet, untagged and -tags integration tests) passes, and an independent read of the 0e4610242..HEAD diff finds no correctness defect against cards 1-6."
---

# Webster round 1 — Bouncer verdict

## Review range

`_lyx/shed/batten-open-ide/status.json` names `product.parent` as `main`.
The merge base is `0e4610242`, and the range holds 14 commits: the six card commits and the eight fixer commits.

## The report's findings

The report's own verdict was `APPROVED` with six LOW and two NIT findings, and the fixer fixed all eight.
I checked each fix on HEAD:

- **F1–F4 (comment line breaks):** the new doc comments on `ExcludeAnchoredDir`, `PathTracked`, `WriteConfig` and the two exclude tests no longer carry fixed-column wraps.
  Every new or changed comment line in the diff now ends at a sentence or clause boundary, as `CONSTRAINTS.md`'s Comment Line-Break Convention requires.
  One small residue remains: `ExcludeAnchoredDir`'s first sentence breaks before ", and returns …".
  That is a compound predicate rather than an independent clause, but it is a clause-level semantic break, not a mid-phrase wrap.
  It does not reach the bar of a finding.
- **F5:** `TestInnerRun_OpenErrorIsWarnedNotEscalated` now counts every `level=WARN` line.
  It asserts that the failed open adds exactly one warning over the success case, and that exactly one warning carries the error text and names the slug.
  This closes the gap the report named.
- **F6:** `assertSingleAttachTask` checks the label, args `reed attach` and `runOn: folderOpen`, and both SpawnDriven hub tests use it.
- **F7:** the comment now reads "a once-marker".
- **F8:** the already-ignored test stages and commits its `.gitignore` before the call.

The report has no `## Focus departures` section, so there is nothing to ratify or reject.

## Independent read

- **innerrun.go:** the ide-opened marker is cleared only under `!found`.
  `openIDEOnce` runs after `recordSpawnConfirmed` at both spawn sites, and before the post-spawn `ReadStatus`.
  Both failed-spawn paths return before it.
  The marker is written whatever the open's outcome.
  `NewInnerRun` resolves a nil `OpenIDE` to a no-op, so every existing constructor behaves as before.
- **SpawnDriven:** the tracked check runs before any write, and the exclude runs before `WriteConfig`.
  `gitignore.Ensure` is never reached on the attach-only path.
  `resolveSpawnTarget` logs the prime-resolution warning once, and `Spawn`'s `primeResolved && slug == primeName` matches its old `primeErr == nil` condition.
- **ExcludeAnchoredDir:** the call goes through the checked `gitexec.Run`, and the `errors.As` plus `ExitCode == 1` handling matches `gitrepo`'s existing use of the same contract.
  The rewrite keeps every existing line and adds a missing final newline.
- **wire.go:** the closure is resolved on each Call, confirms the pair through `taskWorktreeLocation`, and passes the prime location.
  `vscode.Launch` uses `cmd.Start`, so the open never blocks the row.

## Per-card mechanical check

No card carries a `Rename`, `Delete` or `Move` group.
The Create obligation holds.
`PathTracked` has no equivalent: `status.go`'s and `weftgit.go`'s `ls-files` scans serve other queries, and `gitrepo.CommitEmpty`'s `ls-files --cached` checks only for an empty index.
`ExcludeAnchoredDir` has no equivalent among the existing exclude writers.
The Edit groups' blast radius checks out.
`WriteConfig` has two production callers, both in `spawn.go`, and both pass a chain.
`OpenIDE` is optional through its nil default.
`Spawn`'s signature is unchanged.
The plan's `verify:` block passes on HEAD: 13 packages report `ok`, and the whole block exits 0.

Docs: `docs/overview.md` carries both the batten and the ide entries card 6 specifies.
`manifest/designs/` has no batten or ide module doc to update.
