---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed-differently
    reason: "The ENOTDIR read and the ThePathIsARegularFile row are as proposed. The unreadable entry in TestPruneLedger uses a name past the file system's length limit instead of a directory made untraversable with permkit.Chmod: mode bits do not bind root (internal/websterengine/rebaseline_test.go skipAsRoot) or Windows (internal/configengine/config_unix_test.go build tag), so the proposed form would need a skip over the whole of TestPruneLedger or a conditional entry."
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed-differently
    reason: "The proposed sentence names only a missing path and a leftover directory, and after F2 a file at the path also reads as gone. The doc now says the worktree no longer stands, holding no .git entry."
  - id: F5
    disposition: fixed-differently
    reason: "The false premise is gone and PairWorktreeStands is named as proposed. The proposed contrast says the .git stat runs no git, which does not separate the two: fabricengine.List reads the listing through dotgit.ListWorktrees and runs git only on dotgit.ErrRefStorage (internal/fabricengine/worktreelist.go:27-31). The bullet says the stat needs no prime location and no listing."
---

# Fixer report, round 1

## What changed

- **F1** (`_lym/landing/summary.md`, `## Merge danger`): the door is `one-way`, the blast radius names the push ledger and the `TargetOf`, `TargetRefused` and `Start` callers in beamcli, battencli and loomcli, and the entry deletion is described.
- **F2** (commit `949b33a76`): `worktreeGone` reads ENOTDIR as gone.
  `removals` gained `ThePathIsARegularFile`, and `TestPruneLedger` gained an entry whose worktree cannot be read, expected kept with its failure returned.
  The package docs that listed the gone shapes now say a worktree stands while it holds its `.git` entry.
- **F3** (commit `b82ea6e9a`): `pairOf` judges standing through `worktreeGone`.
- **F4** (commit `6b92f8551`): the `WorktreeGone` field doc.
- **F5** (`_lym/landing/summary.md`, `## Design decisions`): the `.git` bullet is rewritten.

The summary's `## What changes` and `## Tests` also describe the F2 and F3 behaviour.
`_lym` files are not committed.

Verification: `lym gate test ./internal/runpusher` passed after the self-check fixes (70 passed, 0 failed, 2 skipped).
The 2 skipped tests are the `TestNewDeps_*` ones, which passed under `--tier 1` before the self-check's comment-only changes.

## Self-check

All four lenses returned a report.

- fix-code-correctness: no findings.
- fix-proposal: no findings.
- fix-consistency, LOW, `## Tests` still says "both removals": fixed.
- fix-consistency, LOW, `canPush` doc still says "exists": fixed.
- fix-consistency, LOW, summary says the old check newly reads a missing path as gone: fixed.
- fix-consistency, LOW, `removals` doc does not fit the regular-file row: fixed, with the `TestTargetOf_...` doc.
- fix-affected-set, LOW, `## Tests` still says "both removals": fixed (same as fix-consistency's).
- fix-affected-set, LOW, `PruneLedger` summary bullet omits a file at the path: fixed.
- fix-affected-set, LOW, `removals` and `TestTargetOf_...` docs: fixed (same as fix-consistency's).

## Disputed

## Deferred

## Rulings
