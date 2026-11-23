---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F12
    disposition: fixed
---

## Changes

- F1: a verification entry now carries a required `finding` on a closed entry.
  `parseVerification` returns each closed key with its finding id, and `closeVerifiedBlocking` confirms a key only when its finding id is one `fixedBlockingFindings` returned.
  The verify stencil's mapping step, output block and rules name the field.
- F2, F3, F5, F6: `TestBouncer_FixVerification` gained rows for a partial close, a key closed for a disputed finding, a verifier spawn error, an unwritable ledger rewrite, a fixed-differently finding, a `spawned: false` report, a MEDIUM finding reported fixed (its id absent from the prompt), an on-disk verification of another round, an empty key and a closed entry with no finding.
  The unwritable rewrite is a directory at the ledger's `.tmp` sibling, not a chmod: the escalation record shares the run directory, and the directory trick also holds for root.
- F4: every row compares the whole parsed round-2 ledger against a literal `ledgerFile`.
- F8: `markLedgerResolved` writes `<ledger>.tmp` and renames it over the ledger, removing the temporary file when the rename fails.
- F12: `parseVerification` refuses a key listed twice.
- F7: the hard-cap halt in `settleDecisionAccept` archives the verification with the judge outputs, and its doc comment says so.
- F9: `rejudge`'s doc says it writes none of those files.
- F10: `pattern/PATTERN-sole-parsers.md` lists the verification file and `parseVerification`.

`lym gate test ./internal/shedadapters ./contracts/stencils` passed before the self-check.
The doc-comment edit after the self-check changed no code.
A later gate repair re-broke three comment lines at sentence boundaries in `bouncer_verify_test.go` and `bouncerverify.go`; no code changed.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: LOW, the `settleDecisionAccept` doc comment still named only the judge outputs; fixed as proposed.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

F11 is a `decision`-class finding held by the run's parent; the fixer changed nothing for it.

## Rulings

None reached this round.
