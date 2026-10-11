---
verdict: APPROVED
---

No defects found in the range `git diff $(git merge-base main HEAD)..HEAD` (no `_lyx/loom/rework` round exists, `product.parent` is `main`, Publish failure is `none`).

Each focus question, settled against code I read and tests I ran:

- **Cards 2, 3, 6, headroom cut.**
  `pattern.Headroom` (`internal/pattern/headroom.go`) cuts `len(head) - max(len(base), HeadroomBytes)` and returns false unless the head exceeds both base and line.
  `TestHeadroom` pins the base-above-line row.
  `patternlint.Lint` is the one function behind the `lint-pattern` verb (`internal/loomcli/lintpattern.go`) and the round gate (`loomshed.NewVerifyGate`), with the same empty-base and base-vs-HEAD logic, CRLF normalised.
- **Cards 5, 6, wholeDiff flag.**
  Only `wireDarn` sets `env.VerifyWholeDiff = true`.
  `wire` sets only the merge base reader, and only when a stat finds the origin record, so wire still writes nothing.
  `NewVerifyGate` selects the form from `wholeDiff` alone, requires a reader for it, and runs the headroom lint after the guard scan and before the comment lint and any test.
  `TestWire_DarnRecipe` covers the loom, record-bearing loom and darn seeds.
- **Card 4.**
  `cardGateCommand` appends `lyx loom lint-comments` then `lyx loom lint-pattern`.
  `lint-pattern` is in the lightweight-wiring verb set, so a broken module config does not refuse it.
- **Cards 8, 9.**
  `ProcPPID`, `ProcState` and `ProcComm` share `procStatFields`, which counts fields after the last `)`.
  `TestProcProbes_OwnProcess` checks them against the test process and an absent pid.
  `grep -rl --include='*_test.go' '"/proc'` outside `internal/testkit/tmuxkit/` is empty.
- **Cards 1, 10.**
  `gofmt -l .` prints nothing.
  `PATTERN.md` is 14725 bytes, under 15000 and under `HeadroomBytes` (15564).
  The moved detail appears as additions in the `pattern/` background files.
- **Card 7.**
  Both budget failures name the row value to set in `cmd/lyx/testdata/test-budget.yaml`.
  The budget rows moved by exactly the new top-level tests (`cmd/lyx`, `internal/pattern`, `internal/testkit/tmuxkit`).
- **Test economy.**
  The new tests each cover a distinct behavior, and the changes to existing tests are extensions of the tests that already covered that behavior.
  `internal/patternlint` has no tests of its own and is covered through `TestLintCommentsCmd_OverARepository` and the verify-gate round scenario, so there is no per-helper test.
- **Docs.**
  `internal/patternlint/doc.go`, `internal/pattern/doc.go`, `internal/loomshed/doc.go`, `docs/overview.md` and `pattern/PATTERN-gate-self-check-parity.md` reflect the verb, lint and gate change.

Verification run: `lyx gate test` over the touched packages and `lyx gate test --tags integration` over `loomshed`, `loomcli`, `pairteardown`, `gatecli`, `shedverbs`, `webstercli` and `tmuxkit`, all green.
