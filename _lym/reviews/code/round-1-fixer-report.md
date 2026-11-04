# Fixer-report, code review round 1

All seven findings are fixed as proposed; none is disputed or deferred.
`lym gate test` is green for `./internal/vscode` (tier 0) and `./internal/ideengine` (tier 1), and for `./cmd/lym` (tier 0).

## Changes

- **F1** `pattern/PATTERN-config-layers.md`: `ideconfig` added to the degrading callers after `hibernateconfig`, matching `configStrictnessDegradingSet`.
- **F2** `spawn_hub_integration_test.go`: the task rows of `checkTaskSlugOpensBareFolder` and `checkWindowCommandFollowsALaunch` list `{beam}.md` and assert Spawn and OpenTask pass the resolved path for a task forked from the prime.
  The beam check forks a task from the beam, which it removes on cleanup because the hub workspace check lists every pair, and asserts OpenTask and Spawn resolve `{beam}` to the beam's slug.
- **F3** `checkTaskSlugOpensBareFolder`: an absolute `open_files` entry makes Seed, Spawn and OpenTask fail with the load error, launch nothing and leave no `settings.json`.
- **F4** `checkPrimeWritesHubWorkspace`: the prime list holds an existing file, an absent file and a `{beam}` entry, and the launch's files equal the resolved paths in order.
- **F5** a directory entry (`docs`) in the beam list is skipped; a task whose prime name cannot be resolved (`checkPrimeNameFailureOpensBareFolder`) and one whose origin record cannot be read (`checkBeamWithoutReadableOriginOpensBareFolder`) open nothing for a `{beam}` entry, beside a file named `.md` that an unskipped empty replacement would open.
- **F6** `applyMarkdownPreview` leaves a present, non-map value unchanged in both paths, and treats a `null` value as absent; rows for the array form with render on and off and for `null` are in `TestWriteConfigOwnsMarkdownPreviewEntry`.
  The package doc, the `WriteConfig` doc and `docs/overview.md` state the array-form exception.
- **F7** `seed` and `OpenTask` load `ide.yaml` over `fabricengine.ConfigDirs(own)`; their doc comments say so, and `placeWindow` stays on `l`'s stack.
  `checkWindowCommandFollowsALaunch` writes a local override into the task's anchor and asserts OpenTask, run from the prime's location, opens the file it lists.
- Test support: `setIDEConfig`'s cleanup tolerates an already-removed file so a check can set it twice; `writeFileUnder` removes its file on cleanup; `setLocalIDEConfig` writes a worktree's local override.

None of these differs from its proposed fix in substance; F5's unresolved-prime row uses a failed prime-name lookup, and the unreadable-parent branch has its own row through the unreadable origin record.

## Self-check

- fix-code-correctness, LOW: `null` association value left unrendered by the F6 guard. Fixed as proposed: `null` now reads as absent, with a row.
- fix-code-correctness, LOW: the prime-name-failure rows pass with the skip branch removed. Fixed as proposed: a `.md` file is created beside the entry, in that check and in the unreadable-origin check.
- fix-code-correctness, LOW: no test fails if F7 reverts to `ConfigDirs(l)`. Fixed as proposed: a local-override row in `checkWindowCommandFollowsALaunch`.
- fix-consistency, LOW: array-form exception missing from the `WriteConfig` doc, the package doc and `docs/overview.md`. Fixed as proposed in all three.
- fix-proposal, LOW: `null` value kept by the F6 guard. Duplicate of the first correctness finding; fixed with it.
- fix-proposal, no finding on F7; its coverage observation duplicates the correctness finding above and is fixed with it.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
