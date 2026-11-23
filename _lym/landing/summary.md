# Bouncer: verify fixed BLOCKING findings before asking the parent

A run whose budget is spent over an open BLOCKING ledger entry no longer escalates to the parent when the fixer fixed that finding and a short check confirms it.
Four darn runs on 2026-11-23 stopped at Code-Bouncer with cause `budget` after round 1, though the fixer had fixed every finding and passed its gates.
Nothing re-reviewed the fix, so the ledger kept the entry `open` and the orch had to read the fixer report and accept each time.

## What changes

With `blocking_escalates` set, the Bouncer now runs a verifier pass before it escalates a spent budget.
The pass runs on the judge's model and checks only the open BLOCKING entries whose findings the round's fixer reported `fixed` or `fixed differently`, against the fix commits.
It writes `round-N-verification.md`, which marks each key `closed` or `open`.
The Bouncer marks each key the verifier closes `resolved` in the round's ledger, and the escalation names only the keys left open.
When none is left, the round is judged as if the ledger held no open BLOCKING entry.

The new code is in `internal/shedadapters/bouncerverify.go`, with a `bouncer-template-verify` stencil for the prompt.
`bouncer.go` calls it from `blockingEscalationDetail`.
The darn, sampler and tapestry recipes, `shed-recipe-spec.md`, `docs/overview.md` and `pattern/PATTERN-sole-parsers.md` describe it.

## Design decisions

- **Every failure asks the parent.**
  A missing or unparseable fixer report or review, a verifier that fails or writes a malformed file, and a ledger rewrite that fails all leave every key open.
  The escalation detail then carries a note saying why the check did not run.
- **A key closes only on a named finding.**
  The verifier must name the review finding id for each closed key, and that id must be one the fixer reported fixed.
  A fixer report written for a fixer that never spawned counts for nothing.
- **No check when the fix changed nothing.**
  When the round's snapshot record shows the review target unchanged by the fix, the verifier does not run.
- **The ledger rewrite is atomic.**
  `markLedgerResolved` writes a temporary sibling and renames it over the ledger.
  `renderLedger` is new in `bouncerfiles.go`, and the ledger's optional fields are now `omitempty`.
- **The verification file is retired with the judge's outputs.**
  A re-judge and a held decision accept archive it, since it ruled on the retired ledger's keys.
- **An existing verification is reused.**
  A readable verification on disk is used as it stands, and a live verifier is attached to instead of respawned.
- **`below-window-accept-blocking` needs no code change.**
  A decision accept below the soft-cap window already passes nothing over an open BLOCKING entry of another class and starts another round.
  Inside the window the accept is retired and the round escalates with cause `budget`.
  A parent ask below the window would also go against this task's aim of asking the parent only after the fixer and the check.
  Only the doc comment in `bouncerdecision.go` changes.

## Scope

The diff covers the main entry and settles `below-window-accept-blocking`.
It does not touch the other three board entries, so each is still open:
`budget-escalation-untested` (`internal/shedadapters/bouncer_escalation_test.go`),
`code-auto-accept-flag-only` (`internal/loomengine/reinterpret.go`),
and `hardener-focus-wiring-untested` (`internal/burlercli`, `internal/shedadapters/roundcontext.go`).

## To check by hand

- `verifier-closes-unfixed-blocking-key` (design, MEDIUM): in `closeVerifiedBlocking`, check that the verifier cannot close a key whose finding the fixer did not report fixed.
- `verify-test-no-partial-close-row` (design, MEDIUM): `bouncer_verify_test.go` has no row where the verifier closes some keys and leaves others open.
- `verify-test-spawn-error-and-write-failure-rows` (design, MEDIUM): `bouncer_verify_test.go` has no row for a failed verifier spawn or a failed ledger write.
- `F11` (decision, LOW): answered by the `below-window-accept-blocking` decision above, which changes no code.
- The verifier prompt in `contracts/stencils/bouncer/bouncer-template-verify.md` was not run against a real model.

## Merge danger

**Door:** two-way
**Blast radius:** bouncer
