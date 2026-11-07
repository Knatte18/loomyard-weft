```yaml
lens: code-affected-set
outcome: reported
```

Task: affected-set review of the commentlint line-length change.

**Per change, set found:**
- `Finding` gained `Rule` and `String()`. Its consumers are `loomcli/lintcomments.go`, `gatecli/failurereport.go` and `loomshed/verifygate.go`, and all three render through `String()`. The tests asserting the rendered text are the two integration tests the diff updates and `verifygate_integration_test.go:233`, which asserts only a substring (`a/wrapped.go:3`) and stays valid. `failurereport_test.go:198-201` uses synthetic findings in the old form. That is a stub input, not output asserted from `Lint`, so it is not wrong. Covered.
- The lint's scope widened to line length. These places still describe the old scope:
  - `internal/gateslot/template.yaml:17` still says "comments (fixed-column-wrapped breaks in // comments)". The diff updated the `LintComments` doc in `tiers.go` but not this shipped template text.
  - `internal/websterengine/doc.go:119` still calls it "the comment line-break lint".
- The way-forward text, the `lint-comments` Short and Long, the error message, `docs/overview.md` (lines 399 and 1144), `doc.go` and the `LintComments` doc comment are all updated. Covered.
- `pattern/PATTERN-gate-slot.md:60` and `PATTERN.md` name only `commentlint.Lint` and need no change. The directive declares no PATTERN changes. No commit carries a `Pattern-Departure:` trailer, and none is needed. Covered.
- Stencils: `webster-body-implementer.md:97` only names the verb, so it is unaffected.

**Findings**

1. LOW, consistency, `internal/gateslot/template.yaml:17`. The shipped gate-config comment describes the `comments` lint as fixed-column-wrapped breaks only, but it now also flags over-long lines. Change it to mention both rules.
2. NIT, consistency, `internal/websterengine/doc.go:119`. It calls the lint "the comment line-break lint", which the diff's own `docs/overview.md` wording no longer matches. Change it to "the comment line lint".

**Tool calls**
1. A grep for consumers and descriptions of `commentlint`, `lint-comments` and `LintComments` across the repo. My context lacked every file outside the diff.
2. Read `verifygate_integration_test.go` lines 225-275, and grepped the loomshed tests and `docs/overview.md`, `PATTERN.md` and `pattern/` for mentions. My context lacked the assertions in that test.
