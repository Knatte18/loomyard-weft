# Fixer report — plan, round 3

## Changes

- **F1** (MEDIUM, consistency), fixed as proposed.
  In `_lym/plan/07-test-tier-check.md`, the way-forward string now offers the support-only form right after `//go:build tier0`: "or the word `untagged` when the file holds support only (TestMain, helpers) and carries no build line".
  The no-statement D7 row now asserts that its Detail holds the `untagged` clause.
  The `untagged` passes row now shows that support-only fix passing.
  The Docs paragraph now says the spec row carries the same way-forward string the finding carries.
- **F2** (LOW, scope), fixed as proposed.
  The D7 rows gain a row in the existing table, not a new test function.
  It covers a card creating `internal/x/x_test.go` (tier0) and `internal/x/helper_test.go` (untagged).
  A single Intent line naming both files fails both with `test-tier-unstated`, each Detail naming `tier0` and the own-line clause.
  Splitting the two statements over two Intent lines passes.
- **F4** (LOW, consistency), fixed as proposed.
  Card 7's Docs paragraph names the Long help sentence it rewrites, the one starting "Every mode but --require-approved also runs rename-tool-missing and pattern-changes-mismatch, which read language.yaml and the decision record".
  The rewrite lists `test-tier-unstated` and moves the clause on what they read so it describes only those two checks.
  Cards 8 and 9 (`_lym/plan/08-seam-test-check.md`, `_lym/plan/09-doc-go-check.md`) each add their check name to that sentence, outside the which-read clause.
  The refs stay typed `Edit` on all three cards.
- **Parent decision** "Tier tags are collected as a set" (recorded with `lym loom decision add`), applied.
  Card 7's collecting sentence says the tags are collected as a set of distinct tags, so the same tag stated on several naming lines counts as one.
  The Docs paragraph carries the same rule into the `loom-plan-spec.md` row.
  The D7 rows gain a row where one file is named on two Intent lines that both state `//go:build tier1`, and it passes.

## Self-check

- fix-consistency, LOW: the decision record's D7 way-forward quote (`_lym/discussion/decision-record.md:122`) no longer matches card 7's string after F1. Deferred, see below.
- fix-claims: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings; it left no finding unchecked.

## Disputed

None.

## Deferred

- The fix-consistency LOW from the self-check is deferred because the only sound change is outside my write surface.
  The premise holds: D7's quoted way forward ends at "such as `//go:build tier0`" and lacks the support-only form F1 added to card 7.
  Reverting card 7 to match would bring back F1's defect.
  The sound change is to append the `untagged` support-only clause to D7's quote in `_lym/discussion/decision-record.md`, which is outside my write surface (`_lym/plan` and the round's report files).
  The parent can record it with `lym loom decision add`.
