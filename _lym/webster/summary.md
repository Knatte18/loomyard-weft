# warp-visibility — heddle

All eleven batches recorded `done`; the scope guard found only informational deviations, and no batch reported a `Pattern-Departure:` or a rename.

## Batches

- **01 pattern-entries** (cards 1, 2): the three PATTERN lines and their background files, plus `lymcwd.BoltDirName` as the one declaration of the Bolt directory name, with fabric's and runproc's names derived from it. Deviations: `internal/fabricengine/junctionnames.go`, `internal/lymcwd/anchor.go`.
- **03 fabric-bolt-dir-sweep** (card 3): fabric's Bolt uses, messages and comments moved onto the Bolt names. No deviations.
- **04 fabric-test-bolt-dir-sweep** (card 4): fabric's tests take the name from the constant. No deviations.
- **05 module-bolt-dir-sweep** (card 5): other packages' messages and comments brought in line. No deviations.
- **06 module-test-bolt-dir-sweep** (card 6): other packages' tests take the name from the constant. No deviations.
- **07 module-bolt-dir-uses** (card 7): Bolt-checkout paths moved from `BoardDir` onto `BoltDir`. No deviations.
- **08 stencil-skill-bolt-dir-sweep** (cards 8, 9): stencils and skills cleared of the literal, the single-declaration guard added, and the `warp_exclude` key added and seeded when fabric wires a warp worktree. Deviations: `internal/boardengine/doc.go`, `internal/fabricengine/warpexclude_integration_test.go`.
- **10 fabric-worktree-seam** (cards 10, 11, 12): fabric's made and took-in seam, the `internal/entrymerge` package, and the board replay run over it. Deviations: the new `entrymerge` and `worktreeseam` files.
- **13 bolt-replayer-registry** (card 13): the board replay seam replaced by a registry of Bolt replayers and a run ledger, following the recorded decision that `ErrReplayerMissing` arises only when no run ledger is registered. Deviations: files in `internal/boardengine` and `internal/fabricengine`.
- **14 heddle-store** (cards 14–17): `internal/heddleengine` with the store, target table, renderer, worktree seam and Bolt replayer. Deviations: the new `heddleengine` files and `internal/fabricengine/replay_integration_test.go`.
- **18 shuttle-prompt-seam** (cards 18–20): the shuttle passage seam appended in `claudeengine`, the `lym heddle` verbs, and the root pre-run installs. Deviations: `cmd/lym/prerunseams_test.go`, `cmd/lym/sandbox_coverage_test.go` and files in `internal/heddlecli`.

## Scope guard

Informational deviations only, listed per batch above; none made a batch stuck.
