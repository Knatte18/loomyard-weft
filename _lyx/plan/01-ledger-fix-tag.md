# Card 1 — ledger-fix-tag

**Edit:**
- `internal/shedadapters#ledgerEntry`
- `internal/shedadapters#ledgerEntryYAML`
- `internal/shedadapters#parseLedger`
- `internal/shedadapters#TestParseLedger_ClassAndSeverity`
- `contracts/stencils/bouncer/bouncer-template-judge.md`
- `contracts/stencils#wordingClaims`

**Intent:** The judge tags every open ledger entry with a fix kind (D1), and the ledger parser keeps the tag.

`ledgerEntryYAML` gains `Fix string` with tag `yaml:"fix"`, and `ledgerEntry` gains `Fix string`, documented as the judge's fix kind: `mechanical`, `design-wrong`, or empty when the judge wrote none.
`parseLedger` copies the value through verbatim and never refuses it: a missing `fix` key or an unknown value is legal, because D1 reads both as `design-wrong`, a reading the soft-cap rule (card 5) applies.
Update `parseLedger`'s doc comment to say the optional `fix` is kept unvalidated.

`bouncer-template-judge.md`'s ledger section:

- The example ledger's open entry carries `fix: design-wrong` after `severity`.
- A new frontmatter rule: every `status: open` entry also carries `fix`, `design-wrong` or `mechanical`, in every round.
  `design-wrong` means the finding says the design itself was wrong, so its fix changes a decision, an approach, a contract or an interface, and that new text needs a fresh review.
  `mechanical` means a local correction whose result the finding text already fixes, so the fix can be accepted unreviewed.
  Choose it from the review's finding text alone: never open the artifacts or the fixer's diff.
  A `status: resolved` entry may omit it.
- The stencil gains no marker and no Go-held variant, and stays free of `{{if}}`/`{{range}}`.
  These two meanings are the ones the escalation brief (card 3) and the arbiter stencil (card 9) point at instead of restating.

`wordingClaims`' `bouncer-template-judge.md` row gains must-claims for `` `fix` ``, `` `design-wrong` ``, `` `mechanical` `` and "from the review's finding text alone", each with a why naming D1.

`TestParseLedger_ClassAndSeverity`'s table gains a `wantFix string` field, and its success check compares the parsed entry's `Fix` to it beside class and severity, so the existing rows assert an empty `Fix`.
It gains rows: an open entry with `fix: mechanical` parses with `Fix` `mechanical`, and one with `fix: sideways` parses without error and keeps `sideways`.

**ImpactSummary:** One new optional ledger field read by nothing yet; the judge stencil's wording and its claims row change, with no marker change, so every existing ledger still parses.

**Commit:** 1: judge tags each open ledger entry mechanical or design-wrong
