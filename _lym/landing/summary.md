# Check fixed BLOCKING findings before a spent budget asks the parent

A darn's Code-Review budget is spent after one round, and the judge never re-reviews a fix.
So a BLOCKING finding that the fixer fixed, committed and gated stayed `open` in the ledger.
`blocking_escalates` then handed the round to the parent with cause `budget`.
That happened in four darn runs on 2026-11-23, and the orch accepted each one by hand after reading the fixer report and the commit.

## What changes

- **A verifier pass runs before the budget escalation.**
  In `internal/shedadapters/bouncerverify.go`, `blockingEscalationDetail` first calls `closeVerifiedBlocking` on the round's open BLOCKING ledger keys.
  The verifier runs when the round's fixer report marks a BLOCKING finding `fixed` or `fixed-differently`.
  It also needs a round with no escalation record yet, and a fix that changed the review target according to the round's snapshot record.
  When all three hold, one verifier pass runs on the judge's model and config role, with role `bouncer-verify`.
  Its prompt is the new stencil `bouncer-template-verify`.
  It reads the listed ledger keys, the findings the fixer reported fixed and the fix commits, and writes `round-<N>-verification.md`, marking each key `closed` or `open`.
- **A confirmed entry is closed in the ledger.**
  Go marks each key that the verifier closed, and that it was asked about, `resolved` in the round's ledger, using the new `renderLedger`.
  The auto-accept, the soft-cap rule and the carry-over then see it closed.
  Only the keys still open reach the parent. If the check fails (no file, or a file that does not parse), every key stays open and the detail says so.
- **A fixer report counts as a green gate.**
  A failed fixer gate archives the round's review, fixer report and acceptance record, so a judged round's fixer report exists only after the gate passed.
- **Re-calls and re-judges stay consistent.**
  A re-call over a recorded escalation spawns no verifier, and a verification already on disk is reused.
  `rejudge` archives the verification along with the judge's outputs, because it ruled on the retired ledger's keys.
- The stencil is registered in `contracts/stencils/stencils.go`, both in the registry and in the role-to-opening-stencil map.
  The verifier spawn has a parity row in `internal/shuttleparity`.
  The shed-recipe spec, `docs/overview.md`, the package doc and the three recipes' `blocking_escalates` comments describe the check.

## Design decisions

- The check lives in the Bouncer at the point where it would escalate, not in the judge.
  The judge stays a sit-rep reader that never opens the fixer's diff.
  The verifier costs one short pass, and only when a spent budget meets a finding the fixer reported fixed.
- Go rewrites the judge's ledger and adds no separate record for the readers to subtract.
  As a result, every reader of the ledger (the soft-cap rule, the carry-over, the next round's judge) sees the same closed entry.
- The verifier reuses the judge's model and `bouncer-judge` config role, so it adds no new model setting and no new agents.yaml role.
- It runs only under `blocking_escalates`, which only the Code-Review rows set. No new recipe key turns it on.

## The other board entries folded into this task

- `below-window-accept-blocking`: no change. `holdAcceptOverOpenEntries` already holds a decision accept when the ledger has an open BLOCKING entry of another class.
  Below the soft-cap window it runs another round, and inside the window it escalates with cause `budget`.
  The `TestBouncer_AutoAccept` row "an accept over a decision finding below the budget leaves an open BLOCKING entry to the next round" pins this. The run cannot land past the entry.
- `budget-escalation-untested`: no change. The `TestBouncer_AutoAccept` row "BlockingEscalates hands a decision finding over an open BLOCKING entry to the parent as a budget escalation" asserts the appended detail "the review also holds decision findings (D1)".
- `code-auto-accept-flag-only`: a flag is enough. `code_auto_accept: true` still auto-accepts every round without an open BLOCKING entry.
  The narrower meaning comes from the recipe rows' `blocking_escalates`, not from the hub value, so there is nothing in the hub layers to rewrite.
- `hardener-focus-wiring-untested`: no change. The wiring is covered end to end.
  `TestBurlerReviewEntry`'s "HardenRoundSetsTheFocusFindingsPermission" covers the recipe row through the producer, round context and round check.
  `TestValidateReviewVerb_RoundChecks` covers the `lym burler validate` path for the hardener and code run directories.

## Tests

`TestBouncer_FixVerification` in `internal/shedadapters/bouncer_verify_test.go` covers these cases:

- a confirmed fix that closes its entry, so the round accepts;
- a fix the verifier keeps open, a verifier that writes nothing, and an unknown status, each of which escalates with cause `budget`;
- a verification already on disk, which is reused;
- a disputed finding, a round with no fixer report, and an unchanged target, none of which spawns a verifier;
- a decision finding beside a confirmed fix, which goes to the decision escalation.

It also checks the verifier's spec and prompt, and that a re-call spawns no further pass.

## Merge danger

**Door:** two-way
**Blast radius:** shedadapters
