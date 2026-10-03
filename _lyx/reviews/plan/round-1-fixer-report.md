# Fixer report — plan round 1

All eleven findings are fixed.
`lyx loom validate-plan` passes on the edited plan with informational findings only.

## Renumbering

The F4 split turned card 14 into three cards, and the later cards were renumbered:

| Old | New |
|---|---|
| 14 webster-verify-gate | 14 verify-gate-report, 15 webster-verify-gate, 16 verify-fixer-fork |
| 15 retire-integration-stage | 17 retire-integration-stage |
| 16 webster-verify-verb | 18 webster-verify-verb |
| 17 docs-and-specs | 19 docs-and-specs |

Every cross-reference was updated (cards 2, 7, 10 and 12, the Card Index, the overview's API line, and the card titles).

## What changed per finding

- **F1 (BLOCKING):** card 3 now targets `internal/loomcli/integration_driverbootstrap_test.go`.
  Its stub reads the `prompt.md` path from the pointer in `$1` and greps the drive-report path from that file.
- **F2:** card 3's Intent states what the removed guard refused before, and that nothing at launch bounds prompt size now.
  Only the session's paged read of `prompt.md` does, and the launch line stays bounded.
  `driverprompt.go` and `driverprompt_test.go` joined the card, so they stop citing the launch cap.
- **F3:** the Webster-Burler closure is now the exported `loomshed.NewVerifyGate` (card 8: new `internal/loomshed/verifygate.go`, `testmain_test.go` and an integration test; the shedrecipe integration test and testmain are no longer created).
  The webster gate gets the exported constructor `websterengine.NewVerifyGate` (card 15).
  Card 18 adds `internal/webstercli/verify_parity_integration_test.go`.
  It runs a three-way verdict-parity test over the verb and both closures, for four cases: pass, failing command, dirty tree, and no `## verify:`.
  The source scan stays as D16's tripwire.
  The overview gains a "Gate parity" shared decision.
- **F4:** card 14 is split into three cards:
  - 14: the report, the findings text and the card hint (D4).
  - 15: the gate, the commit check, the outcome mapping and the loomcli `ParentBranch` wiring (D3).
  - 16: the fixer stencil, Merriam's gate-failure section, the `forkOwnReport` remap and the record-batch skip (D5).

  `verify_gate_attempts` stays in card 10: the review's verdict on that placement was "not a defect".
- **F5:** card 17 no longer deletes `accumulatedCardSHAs`.
  Card 14's hint reads it and updates its doc comment.
  `verifygatereport.go` and its test joined card 17 for the `VerifyFailure` rename.
- **F6:** card 15 records the pre-fix head at the first failed evaluation of any kind, dirty or verify.
  It runs the commit check whenever a pre-fix head is recorded, and a test pins the dirty case.
- **F7:** card 15 targets `internal/loomcli/wiring_test.go`, with a case asserting the wired `RunDeps.ParentBranch`.
- **F8:** card 15 states that `StartMaster` is Merriam's only start: reclaim stops the old strand and `Run` starts a fresh one.
  `Run` now refuses a `deps.Gate` that already names `verify`, and a test pins the refusal.
  `internal/shedrecipe/entries_simple.go` joined card 15 to update `websterEntry`'s doc.
- **F9:** the ImpactSummaries of cards 13, 17 and 18 now state a reach conclusion.
  So do the new or rewritten cards 3, 8, 14, 15 and 16.
- **F10:** card 19 adds `contracts/specs/final-summary-spec.md` and names `lyx webster verify` in the overview's webster entry.
- **F11:** card 3 drops `ReadFile` from the `Shell` interface and `posix.go`.
  pwsh keeps it as an internal helper for `Source`, and `shell_test.go` and `command.go`'s header follow.

## Correction made while fixing

The first draft of card 14 called its hint test tier 1 while spawning git.
That violates the Test Tier Purity Invariant, which bars git spawns from untagged test files.
`cardHint` now takes a `changedPaths` seam, so the unit test is tier 1, and card 15's runlevel test covers the real path end to end.

## Deferred

None.
