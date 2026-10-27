# Fixer-report — code review round 1

## Changes

- **F1** (fixed as proposed): `TestLoadState_UnusualFiles` asserts, right after `LoadState`, that the scratch directory still does not exist, in every row including the absent-file row.
  Commit `6e3bcebc`.
- **F2** (fixed differently): the proposed `.weft`-absent assertion in the `beforeWire` hook cannot hold, because Add creates `.weft` before the hook fires (the first run failed on it).
  The hook now asserts that `.weft` existence and `info/exclude` are unchanged by the read.
  The review's named alternative is added as `TestReadOrigin_CreatesNoWeftDirectoryAndSeedsNoExclude`: it removes `.weft` and empties the exclude, calls `ReadOrigin`, and asserts neither came back.
  The exclude was emptied after the correctness fork showed that comparing an exclude Add had already seeded could not fail.
  Commits `68890574`, then the follow-up on the exclude check.
- **F3** (fixed as proposed): `TestReadJSON` row `LockUnopenableForAnotherReason` puts a directory at the lock path and asserts an error is returned.
  Commit `f9748ec6`.
- **F4** (fixed as proposed): `TestInWorktreeStaging` in `internal/fabricengine/destroy_test.go` covers a nested path, the staging directory itself, a prefix-only name, a prefix inside a name and an unrelated path.
  `TestHubOrchs_ListsWiredBeamsAndSkipsUnwiredPairs` captures the logger, resets it before `HubOrchs` and asserts no `level=WARN`; it is no longer parallel because the capture swaps the global logger.
  Commit `d221c07e`.
- **F5** (fixed as proposed): `VerifySeedOwnership` no longer calls `MkdirAll`, and the `path/filepath` import went with it.
  Commit `876c7ac0`.

- **Gate repair**: the header comment of `internal/fabricengine/add_hubreader_integration_test.go` that F2 widened was wrapped mid-sentence, which stopped the gate before any test ran.
  It now has one sentence per line.
  `lym gate test --upto 1 ./internal/fabricengine` passes.

## Self-check

- fix-code-correctness, MEDIUM: the exclude assertion in the new `ReadOrigin` test cannot fail, since Add already seeded it. Fixed with my own change (empty the exclude first); the `beforeWire` hook keeps its unchanged-exclude check, which only guards `.weft`-free changes.
- fix-code-correctness, LOW: the `level=WARN` assertion guards a narrow regression. No change: the review asked for it and the fork found no harm.
- fix-consistency, LOW: twin `MkdirAll`-before-`ReadJSON` sites remain. Merged with the affected-set finding below; deferred.
- fix-affected-set, LOW: eight sites outside `seed.go` still make the lock directory before a status read. Deferred, see below.
- fix-proposal: no findings.

## Disputed

None.

## Deferred

- **F5's twin sites** (`internal/beamcli/deps.go:96`, `internal/loomcli/resume.go:51`, `internal/loomcli/circling.go:222`, `internal/battencli/startguard.go:297`, `internal/battencli/wire.go:656`, `internal/tendercli/deps.go:297` and `:607`, `internal/hibernatecli/deps.go:128`).
  The review scoped F5 to `VerifySeedOwnership` and rated the race unreachable from the hub reader; these sites are in other packages and the forks found comments and failure branches in `resume.go`, `wire.go` and `circling.go` that depend on the mkdir.
  Removing them is a wider change than the finding and needs the orch's decision on whether the board entry's "a read never creates a directory, anywhere" covers them.
