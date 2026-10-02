# Webster-Burler round 1 — fixer report

All three findings are fixed, each in its own commit, and none was pushed.
After the fixes, `go build ./...` and `go test ./...` pass.

## Fixed

- **F1 (LOW)**, commit `c0d7f01dd`.
  `clone`'s `Use` string, its mirror comment in `internal/fabriccli/fabric.go` and the usage refusal in `internal/fabriccli/clone.go` now start with `[--shortname <shortname>]`.
  The help's first example, the two-argument first bind, now passes `--shortname mono`.
  The usage-refusal test matches on the `usage: lyx fabric clone` prefix and still passes.
- **F2 (LOW)**, commit `b89ef2c83`.
  README's Getting started clone line now passes `--shortname <shortname>`, with the block's comment column realigned.
  `docs/sandbox-hub.md` step 4 now shows the `--shortname lyt` the sandbox tool passes.
  Fabric suite scenario F13 now clones the empty weft with `--shortname <shortname>` and says a first-ever bind refuses a clone without one.
  The "it always did" note is gone.
- **F3 (NIT)**, commit `bba52bb95`.
  `docs/overview.md`'s fabric bullet is now one sentence per line, with further breaks at independent-clause boundaries.
  No wording changed.

## Deferred

None.
