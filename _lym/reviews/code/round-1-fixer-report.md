# Fixer report, code round 1

## Changes

- **F1** (commit `69736ff1`): `internal/loomcli/sharedbootstrap.go` reads `fabricengine.EnvSyncOptions()` once as `syncOptions`, skips the `AnchoredPathHasContent` probe when `SkipGit` is set, and passes the same `syncOptions` to `CommitAnchoredPaths`. Fixed as proposed.
- **F2** (commit `5039422d`, plus the doc-comment edits to `sharedbootstrap.go` that landed in the F1 commit): the `seedAndCommitBootstrap` doc lists the friction directory among the committed paths, the `ensureFrictionDirAfterSeed` doc says a resume leaves notes on disk and the commit step still commits them, and `docs/overview.md` and `internal/loomcli/doc.go` say every start or step commits what the friction directory holds, with the first seed's clear as the case that matters. Fixed as proposed.
- **F3** (commit `fa06216a`): new `internal/fabricengine/anchoredpathhascontent_test.go` with a table test over a real records worktree (tracked file deleted on disk, untracked file, ignored-only directory, empty directory). The re-entry subtest of `TestSmokeStepBootstrapWiring` now asserts the resume note is tracked after the second step. Fixed as proposed.
- **F4** (commit `be69ce57`): the `TestBootstrapCommitPaths` comment now names the tracked-or-untracked-content condition and the row is `FrictionDirectoryHasContent`. Fixed as proposed.

**Gate repair:** the gate flagged two fixed-column-wrapped comment lines this round added in `internal/loomcli/sharedbootstrap.go` (the `seedAndCommitBootstrap` doc and the `ensureFrictionDirAfterSeed` doc). Both now break at sentence boundaries, one sentence per line, in a follow-up commit.
A second gate pass flagged the first sentence of the `seedAndCommitBootstrap` doc as still one over-long line; it is now split into three sentences, one per line, in a further commit.

A third gate pass failed the fabric-vocabulary enforcement test: the F1 comment in `sharedbootstrap.go` named the weft, which only the fabric owner set may do. It now says "the fabric's git", in a further commit.

Verification: `go vet` on both packages, `lym gate test ./internal/fabricengine --tier 1` (0 failed), `lym gate test ./internal/loomcli --tier 0 --tier 2` (0 failed; `re-entry_keeps_friction_notes` ran and passed).

## Disputed

None.

## Deferred

None.

## Self-check

- fix-code-correctness: no findings. Its LOW open risk, that another part of `step` might also commit the planted note and so make the new assertion pass without the probe, is unverified and names no evidence; no change made. It confirmed the new tests fail when the probe's flags are removed.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

Lens reports are saved beside this file as `round-1-fixer-report.lens-<lens>.md`.
