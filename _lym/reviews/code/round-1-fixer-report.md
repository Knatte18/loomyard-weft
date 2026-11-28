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
    disposition: fixed-differently
    reason: "The proposed rule skips a name that opens a line, which would pass its own required row `max_bounces: must be at least 0`. The name is skipped instead only when the colon is followed by one word or nothing on the line, with a comma or closing brace ending the value only inside an open brace."
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
---

# Fixer report, round 1

## Changes

- **F1:** production comments that still named a key bare now name it in full, including every site the review listed.
  The search covered comment words equal to a template key and the possessive and `the <module> <key>` phrasings.
- **F2:** `_test.go` comments under `cmd/` and `internal/` were swept the same way; test-data literals are unchanged.
- **F1 and F2, left bare on purpose:** recipe row and seed fields, retired board entry fields, drift step fields,
  prompt markers, GitHub's `pull_request` rule, state and envelope fields, YAML examples,
  generic mentions that name no key, and comments where no single module fits.
- **F3:** added `TestKeyNames_DetectorReportsOnlyBareNames`, a table over `bareKeyNames` plus a parsed-source case for `stringLiterals`.
- **F4, F5:** restored the retired entry key wording and the lost words in `internal/dobbyengine/reinterpret.go`.
- **F6 (fixed differently):** added `isDocumentField`; the proposed change not made is the "opens a line or follows `{` or `, `" rule, for the reason above.
  A one-word problem after the colon, such as `timeout_min: %w`, `max_bounces: %v` or `max_bounces: required`, is still skipped,
  since it reads the same as a document value such as `max_bounces: 3`.
- **F7:** each allowlist entry is keyed by file, name and literal.
- **F8:** updated `_lym/landing/summary.md`: coverage, the guard's skip rule, the meaning claim and the blast radius.
- **F9:** added `--unset "` to `qualifiedPrefixes`.
  The commit carries a `Loosened-Check` line.
- **Comment line repair:** comment lines this round added that wrapped mid-sentence or reached 120 characters
  were rewritten as shorter sentences, one per line.
  This includes the two F5 comments in `reinterpret.go`, whose wording now differs from the review's proposed lines
  while keeping the move and what is carried.

A comment sweep of this size was made by one subagent; its diff was checked to hold comment lines only.

Verification: the full `lym gate test` passed over the final tree, after the comment line repair.

## Self-check

- fix-code-correctness, MEDIUM (a comma after the first word hid a bare key): fixed, with a new table row.
- fix-code-correctness, LOW (one-word problems are skipped; state the limit): fixed, stated under F6 above.
- fix-consistency, MEDIUM (`gatecli/cardfixer.go` still said "fixer timeout"): fixed.
- fix-consistency, MEDIUM (`shuttle.engine` named by phrasing): fixed.
- fix-consistency, LOW (`orch.remote_control` named as "the switch"): fixed.
- fix-consistency, LOW (summary's "at most one word" too wide): fixed.
- fix-proposal: no findings.
- fix-affected-set, LOW (four keys pointed at by phrase): fixed, along with the same phrasing in `seatengine/geometry.go` and `burlerengine/fix.go`.

## Disputed

None.

## Deferred

None.

## Rulings

None.
