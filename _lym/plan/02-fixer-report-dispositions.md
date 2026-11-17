# Card 2 — Fixer-report dispositions frontmatter and the fix check

**Create:**
- `internal/burlerengine#Disposition`
- `internal/burlerengine#DispositionFixed`
- `internal/burlerengine#DispositionFixedDifferently`
- `internal/burlerengine#DispositionDisputed`
- `internal/burlerengine#DispositionDeferred`
- `internal/burlerengine#DispositionRuling`
- `internal/burlerengine#FindingDisposition`
- `internal/burlerengine#FixerReport`
- `internal/burlerengine#ParseFixerReport`
- `internal/burlerengine#CheckFixerReport`
- `internal/burlerengine#FixerReportGateEntry`
- `internal/burlerengine#TestCheckFixerReport`

**Edit:**
- `internal/burlerengine#skippedFixerReport`
- `internal/burlerengine#skippedRoutedFixerReport`
- `internal/burlerengine#Profile.fixTable`
- `internal/burlerengine#TestEngine_Fix_TableShapeAndFiles`
- `internal/burlerengine#TestEngine_Fix_Outcomes`
- `contracts/stencils/burler/burler-step-fix.md`
- `internal/burlerengine/doc.go`

**Extends:**
- `pattern/PATTERN-sole-parsers.md`

**Uses:**
- `internal/burlerengine#newFixCase`
- `internal/burlerengine#fakeShuttle`
- `internal/burlerengine#halfScript`

**Intent:**
D1: the fixer-report gains `---`-delimited YAML frontmatter over its prose, and `internal/burlerengine` is its sole parser.

`FindingDisposition` holds `id`, `disposition`, `ground`, `rebuttal`, `evidence` (a list of strings) and `reason`.
`FixerReport` holds `Dispositions []FindingDisposition` (yaml `dispositions`) and `Spawned *bool` (yaml `spawned`), so a present `spawned` key is told apart from an absent one.
`ParseFixerReport` refuses a report with no frontmatter, an unclosed one, invalid YAML, or a disposition outside the five `Disposition` constants.
`ParseFixerReport` refuses a frontmatter holding a top-level key other than `dispositions` and `spawned`, as `checkReviewKeys` does for the review, so a misspelled `dispositions` key never decodes as an empty list (`PATTERN-tolerant-records`: the report is agent-written and stays strict).
Its message names the stray key and the two legal keys, and ends `way forward: rewrite the frontmatter of the fixer-report with only the dispositions and spawned keys`.

`CheckFixerReport` judges a fixer-written report against the accepted review's findings and refuses, each with a message ending `way forward: rewrite the frontmatter of the fixer-report ...` naming what to add or remove:
- a `spawned` key, which only Go writes;
- an entry whose id the review lacks, or an id listed twice;
- a finding the fixer owns with no entry: every finding, but a `decision`-class one when `decisionsRouted` is true;
- `disputed` without `ground` in {`premise false`, `every fix worse`}, without a non-empty `rebuttal`, or with an empty `evidence` list or an item not of the form `<path>:<line>` (a non-empty path, a colon, a positive decimal line);
- `fixed-differently`, `deferred` or `ruling` without a non-empty `reason`;
- `ruling` on a finding that is not `decision`-class, or on any finding when `decisionsRouted` is false.
On a routed round a `decision`-class finding may have no entry or a `ruling` entry.

`FixerReportGateEntry` is named `fixer-report`, has 3 attempts and `PassOnCap: true`; its gate reads the review at `reviewPath` through `ParseCurrentReview` and the report at `reportPath`, and fails with the parse or check error and a line naming the report to rewrite.
`Profile.fixTable` appends it after the wrapped `opts.Gate` entries, unwrapped, built from `p.FixerReportPath`, `p.ReviewPath` and `opts.DecisionsRouted`, so a spawned, probed and resumed fixer are gated alike.
Go-written reports never pass through the table, so the `spawned` refusal never meets one.
On the cap the round goes on as today: the fix half's result is not failed, nothing is archived, and no bounce is spent.

`skippedFixerReport` and `skippedRoutedFixerReport` open with frontmatter `spawned: false` and `dispositions: []` over their present prose.

`burler-step-fix.md`'s fixer-report rule tells the fixer to open the report with that frontmatter: one entry per finding it owns, the keys each disposition needs, evidence as `<path>:<line>` with a dispute resting on verify output citing the test file and line, `ruling` only for a parent ruling on a routed `decision`-class finding, and never a `spawned` key; the prose sections stay below it.
The dispute grounds, disposition order and deferral rule are unchanged.

`PATTERN-sole-parsers.md` gains a bullet and a section for the burler review file (`burlerengine.ParseReview`) and the fixer-report frontmatter (`burlerengine.ParseFixerReport`), naming every consumer as reading their model; the entry line in `PATTERN.md` is unchanged.
doc.go gains a paragraph on the fixer-report format and the fix check, and its skipped-fix sentence ("Fix writes a normal fixer report saying nothing was fixed") is rewritten to say that report opens with `spawned: false` and empty dispositions.

Tests, all tier0:
- `TestCheckFixerReport` is one table over parse and check, one row per refusal above, each asserting the refusal's `way forward:` clause, plus passing rows: a complete report; a routed round whose `decision`-class finding has no entry; a routed round with a `ruling` entry; a review with no findings and empty dispositions.
  An unrouted round missing the `decision`-class finding's entry is refused.
  A frontmatter carrying a misspelled `dispositons` key is refused with the stray-key way forward.
  `skippedFixerReport` and `skippedRoutedFixerReport` each parse with `Spawned` false and no dispositions.
  A row runs `FixerReportGateEntry`'s gate over a review path holding an unparseable review and asserts the failed result names the review's parse error and the report to rewrite.
- `TestEngine_Fix_TableShapeAndFiles` asserts the fix table's last gate entry is `fixer-report` with 3 attempts and `PassOnCap`.
- `TestEngine_Fix_Outcomes` gains rows: a fixer whose report misses a disposition on every attempt ends done with the gate let through, the result not failed; one whose report is fixed by a re-prompt rewrite passes.

**ImpactSummary:** The fix half's gate gains one pass-on-cap entry and the two Go-written reports gain frontmatter; every fix row, probe and resume reads the same table, and no caller's signature changes.
