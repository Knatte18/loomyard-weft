# Round 2 fixer-report — plan

## Changes

- **F1** (MEDIUM, fixed as proposed), `_lym/plan/07-test-tier-check.md`:
  the `test-tier-unstated` way forward now reads "add an Intent line naming the file and its `//go:build` line, such as `//go:build tier0`, on a line that names no other test file; where the lines name several tags, join them with || only for a helper several tiers share, else keep one; drop untagged or the tag when both appear".
  A new Intent sentence says a line that also names another test file carries that file's tags into the file's collection.
  The Docs paragraph's spec row states that rule and the card rule asks for each tier statement on a line that names no other test file.
  `TestValidatePlan_CardChecks` gains the D7 row the fix names: one line naming `a_test.go` and `b_test.go` with `tier0` and `tier1` fails both, each Detail naming both tags and the own-line clause, and the split form passes.
- **F2** (LOW, fixed as proposed), `_lym/plan/09-doc-go-check.md`:
  `internal/loomcli#loomCLI.validatePlanCmd` moved from `**Extends:**` to `**Edit:**`, after `plan:internal/loomshed#TestValidatePlan_CardChecks`.
- Self-check changes, `_lym/plan/07-test-tier-check.md`:
  the pass rule's base-name match now counts only a token of its own, never the tail of a longer path, so two created files sharing a base name each have a passing statement by full path.
  A D7 row pins that case (`internal/x/a_test.go` at `tier0` and `internal/y/a_test.go` at `tier1`, each on its own line, passes).
  The spec row names the token rule.
  The Intent sentence, the spec row and the card rule now all say "another test file" with no "created" or "of the card" qualifier, which matches the check: it collects every tag on a line naming the file.

`lym loom validate-plan` after all changes: `ok: true`, with only the existing informational `create-new-unit` finding on card 4.

## Self-check

- fix-consistency, LOW, the own-line rule is worded three ways ("no other test file", "of the card", "created" / "both" against "all"): fixed as proposed, by making all three use the unqualified wording; merged with fix-proposal's second finding.
- fix-proposal, LOW, base-name matching makes the own-line rule impossible for two created files sharing a base name: fixed as proposed, with the token rule, a spec mention and a D7 row.
- fix-proposal, LOW, the card rule and the Intent sentence qualify the other file as created or the card's own, which the check does not: fixed as proposed, merged with the fix-consistency finding above.
- fix-claims: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
