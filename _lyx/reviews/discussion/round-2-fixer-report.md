# Fixer report — discussion, round 2

## Changes

- F1 (BLOCKING) — fixed as proposed, option (a): the implementer fork runs a card's `**Verify:**` only through a new verb, `lyx gate card-verify <card id>`, which runs the lines through the same slotted runner as Go's re-run (D8).
  D13's first holder names both callers, with the runner's slot and 10-minute timeout, and its GOFLAGS `-trimpath` and `-p` rule covers both.
  The acceptance criterion now says the fork's run of a `go vet` line holds one slot, and Scope and the plan-writer notes name the verb and the stencil lines it changes.
  The declared `PATTERN-gate-slot` text holds unchanged, since the fork's run is now a listed non-tier holder.
- F2 (MEDIUM) — fixed as proposed: D8's Publish failure bullet renders each failing-test run and the package pass as its own `lyx gate test --wait --tier <failing tier>` call joined with `&&`; the verifytree bullet allows such calls joined by `&&`.
- F3 (MEDIUM) — fixed as proposed: the third acceptance criterion names the `go` scoper for the `.md`-only no-run, and says under `none` the listed tiers run whole.
- F4 (MEDIUM) — fixed, first branch: `cmd/testtiming` holds one slot in every mode, prebuild included, and appends `-trimpath` and `-p` to every `go` command (D13); `PATTERN-test-tiers` names every mode; a criterion pins it.
- F5 (LOW) — fixed as proposed: D1 states that a value holding a brace, comma or bracket is double-quoted in the flow mapping, with a quoted `command` in the example; D12's GOFLAGS value is quoted.
- F6 (LOW) — fixed as proposed: Open risks' first bullet names in-flight `**Verify:**` lines holding `--tags`, and the way forward.
- The support log records round 2 and two auto-picks in the question ledger.

## Self-check

- fix-way-forward, BLOCKING (forks are refused every `lyx webster` command, so `lyx webster card-verify` could not run) — fixed: the verb is now `lyx gate card-verify`, and D8 says why.
- fix-way-forward, MEDIUM (editing an in-flight card's Verify needs a rebaseline) — fixed: the Open risks sentence adds `lyx webster rebaseline` naming the card.
- fix-proposal, MEDIUM (the shared runner discards output, so the fork cannot self-fix) — fixed: the verb passes each line's output through, and the criterion says so.
- fix-proposal, LOW (in-flight gate calls lacking `--wait`) — fixed: added to the same Open risks sentence.
- fix-proposal, LOW (the `none` tail ignored D2's covers skip) — fixed: the tail reads "runs the listed tiers whole, skipping a covered one as D2 says".
- fix-consistency, LOW (a rejected alternative still said "one call") — fixed: the support-log bullet allows calls joined by `&&`.
- fix-consistency, LOW (the stencil's foreground rule and `FAILED` wording also speak per line) — fixed: the plan-writer note names them.
- fix-consistency, LOW (D13's rationale only fit `-resources`) — fixed: the rationale names what each mode measures.
- fix-claims, LOW (the recovery twin of step 5 does not exist) — fixed: the note drops it and adds where the runner lives.

## Disputed

None.

## Deferred

None.
