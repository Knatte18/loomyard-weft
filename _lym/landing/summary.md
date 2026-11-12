# Add `lym gate redundant` and the adoptcli coverage-map pilot report

The board's coverage maps exist to cut the test suite, but nothing listed a whole package's redundancy candidates from its map.
This change adds that verb and a pilot report that answers three questions on one package, `internal/adoptcli`:
can a map be built, what does a full set cost, and is the map of any use for cutting tests or scoping the gate.
No test is cut or folded, and no other package's map is built.

## What changes

- `lym gate redundant <package>` reads the package's coverage map and prints its redundancy candidates as JSON.
  Each candidate carries its tier, seconds, covering tests and whether it is in the removable set.
  The answer also lists the keep-marked tests with their reasons and the tests not judged with why,
  and it carries the map's commit, build time and test count.
  It prints `map_build_failure` when a later coverage build failed, and omits the key otherwise.
  It runs no test and takes no gate slot.
  It refuses with a way forward outside a hub, for a missing, unreadable or failed-build-only map, and for a failed keep scan.
- `testcover.JudgeMap` judges every test of a map by the package-scoped verdict, in the map's order.
  A `//testtiming:keep` test gets the new reason `testcover.ReasonKept` and is neither judged nor a cover.
  An entry with a recorded outcome or a spawn-marked entry is not judged either.
- The report is `docs/research/speed/coverage-map-pilot-2026-11-12/report.md`,
  with `packages.tsv` (the scaling input) and `adoptcli-redundant.json` (the verb's answer for the package).
- `docs/overview.md`, `internal/gatecli/doc.go` and `pattern/PATTERN-test-speed.md` describe the verb,
  and `docs/research/README.md` lists the report.

## What the report finds

- Build: one `lym gate testmap internal/adoptcli` call succeeded in 34.3 s wall and 47.8 s CPU, holding one of three gate slots.
- Full set: an estimate of 45 to 165 minutes on one slot, 65 to 230 CPU-minutes.
  The range turns on whether the bookkeeping between test rounds costs per test or per round.
  The hub's upkeep pass had already built the full set in 44 minutes, at the low end.
- Cutting: of 30 judged tests, 2 are candidates, and both assert something their covering tests do not.
  The map is a reading filter, not a verdict.
- Scoping: over two landed commits the selector never left out a test covering the changed code.
  It named no test for one commit and fell back to the whole package for the other.

## Design decisions

- `lym gate redundant` reads the keep markers from the current worktree, not from the map's commit.
  The map is read as the last `lym gate testmap` call left it.
- `JudgeMap` takes the keep set as an argument, so `internal/testcover` stays free of file scanning.
- The verb takes one package and answers in under a second, so a full set costs a reader nothing once built.

## To check

- `TestGateRedundant` has no case for two of the verb's refusals: a map whose test names a block index outside its block table,
  and a package holding a `//testtiming:keep` line with no reason.
  Both branches are in `internal/gatecli/redundant.go`.
- No regression check ran for this task, and the verb was not run against a package other than `internal/adoptcli`.
