---
dispositions:
  - id: R1
    disposition: fixed
  - id: R2
    disposition: fixed
  - id: R3
    disposition: fixed
---

# Round 2 fixer-report

## Changes

- **R1** (fixed as proposed, plus one `TestUpdatePlaces` row):
  - Card 11 changes `reportUpdateFrom`'s signature to take `tilingOutput string` after `snap`.
    The report-update bullets now state the landed desktop and the output for each path.
    After an apply, the window landed on the plan's target and the output is `tilingOutput`.
    Under mode `off`, it landed on the desktop it sits on in the locked snapshot, `""` when it sits on several or on all, and the output is that window's snapshot output.
  - Card 11's `TestFlow_Place` mode `off` row now asserts the entry's `LastDesktop`, `Home` and `Output`.
  - Card 9's `ReportUpdate` parenthesis now states the same per-path rule.
    `updatePlaces`' first rule now says that an empty landed desktop settles the entry and keeps its `LastDesktop` and `Home`.
  - Card 9 gains a `TestUpdatePlaces` row for that empty-landed rule.
    The fix introduces the rule, and no other row covers it.
- **R2** (fixed as proposed):
  - Card 8 adds `TestContract_TheRealKWinAppliesAsRecorded` under Create, holding the `Apply` step with no skip and no flag.
    Its doc comment states that it moves a throwaway window, and the operator runs or approves it by name with the `lym gate test --tier 3 ... -run` line.
  - `-run TestContract_TheRealKWinAnswersAsRecorded` runs the steps that move nothing.
  - The `contract_test.go` sentence names both tests, and the file's `tier3` stays on the line that names the file, as `validate-plan`'s test-tier check requires.
  - The overview's contract-test-tier decision now names the tests in the plural.
- **R3** (fixed as proposed):
  - Card 12's `Spawn` registers the child only when the worktree is not the prime and the caller's environment holds a non-empty `pauserecord.RunProcessEnv`.
    A child opened outside a run is not registered and ends within its own bounds.
    The card's Uses gain `internal/pauserecord#RunProcessEnv` and `internal/pauserecord#RunProcessMarked`.
  - The overview's run-registration decision is rewritten to the same rule, and card 12 keeps its reader analysis for registered children.
  - In `TestPlaceWindowChild`, every row sets the variable with `t.Setenv`.
    `TaskWindow` and `PrimeWindow` mark it, and the two refused rows set it empty.
    A new `UnmarkedTaskWindow` row on a task of its own sets it empty, and asserts that the marker appears and that no `RolePlacement` record exists while the command waits.

`lym loom validate-plan` returns `ok: true` after these changes and after the self-check fixes.

## Self-check

- fix-consistency, LOW: card 6:89 still named the contract test in the singular. Fixed as proposed: it now reads "The contract tests that drive a real KWin are card 8's".
- fix-consistency, LOW: in card 12's `RefusedTaskMode` row, the inserted clause read as the reason no other row resolves its mode. Fixed as proposed: the clause moved after the "so" clause.
- fix-proposal, LOW: a whole-package `tier3` run of `kwinplace` also runs the apply test and moves a window. Fixed with my own change: card 8 now says so beside the two `-run` lines. I added no skip, since a skip is what R2 removed under `PATTERN-test-speed`'s "No test skips by design".
- fix-claims: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
