---
round: 1
ledger:
  - key: fixed-column-wrapped-new-comments
    rounds: [1]
    status: resolved
  - key: tierpurity-gofmt-allowlist-condition
    rounds: [1]
    status: resolved
  - key: gitrepo-messages-name-deleted-oracle-functions
    rounds: [1]
    status: resolved
  - key: gitkit-doc-stale-export-list
    rounds: [1]
    status: resolved
  - key: hubforge-doc-only-exports-incomplete
    rounds: [1]
    status: resolved
---

Round 1 reviewed the 45 card commits from merge base `518dc7400` against `main`.
It raised one MEDIUM and four LOW findings, and the fixer resolved all five in commits `11478faaa` through `639596451`.

- New and changed Go comments had fixed-column mid-phrase breaks, breaking CONSTRAINTS.md's Comment Line-Break Convention.
  They are rejoined, and every wrapped comment left in the range is legacy text moved verbatim.
- `cmd/lyx/tierpurity_test.go` was not gofmt-clean.
  It is fixed, and the whole range is now gofmt-clean.
- gitrepo's parity failure messages named the oracle functions card 17 deleted.
  They now name the `gitoracle` functions.
- gitkit's package doc carried stale four-name export lists.
  It now describes its exports by group.
- hubforge's "only exports" sentence left out `TestShortname`, `SeedConfig` and `SeedFabricConfig`.
  It now lists them.

Card 17's Delete group passes assert-no-callers.
Every test tier passes at HEAD.
No finding remains open.
