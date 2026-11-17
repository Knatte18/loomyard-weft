```yaml
lens: affected-set
outcome: reported
```

Affected-set lens: what each card's change forces outside the card, and which cards cover it.

**Per change: what reads or asserts it, and the covering cards**

1. **Card 2: the Go-written skipped fixer-reports gain frontmatter.** Readers are `internal/burlerengine/fix_test.go:352-354`, which compares the written report with the `skippedFixerReport` and `skippedRoutedFixerReport` vars, so it follows the vars; `burlerengine/doc.go:147`; and the fixer stencil. Outside burlerengine, no production code parses a burler fixer-report: treadleengine keeps its own round files, and landingshed has its own format. Covered by Card 2.
2. **Card 5: `checkReviewKeys` admits `prior` and its refusal text changes.** Readers are `verdict.go:199`, `burlerengine/doc.go:557`, `contracts/stencils/burler/burler-review-format.md:28` and `internal/burlercli/validate.go:32`, the help text "The frontmatter's only top-level keys are verdict and findings". Cards 5 and 6 cover the first three. `validate.go:32` is only partly covered: Card 8 edits the help text but names only the round checks and the git-repository sentence (finding A).
3. **Card 10: the fifth `EscalationCause`.** The texts that list the causes:
   - `shedadapters/escalation.go:3,22,30,37,79` (Card 10)
   - `loomcli/circling.go:43,73,344,357` and `loomcli/doc.go:577` (Card 11)
   - `arbiterstop` (Card 11)
   - `contracts/orchrole/orch-role.md:147` (Card 11)
   - the escalation stencil (Card 10)
   - test consumers `loomcli/circling_test.go`, `loomrecipe/escalation_test.go`, `loomrecipe/arrasrecipe_test.go` and `shedarbiter/call_test.go`, which take no forced edit
   - `shedadapters/doc.go:140-142`: "It does so for one of three causes. A fourth cause, decision…". Card 10 lists that doc.go under `Extends` only, so nothing rewrites the count (finding B).
   - `docs/overview.md:1272-1284`, which lists what the Bouncer escalates for: circling, the hard cap, unreadable inputs. Card 12 edits `docs/overview.md` only for the review budget keys line, `docs/overview.md:1443` (finding C).
4. **Card 9: the parent notice gains a way-forward clause.** Readers are `sheddriver/driverules.go:36-141,260` and `shedengine/status.go:188`. Both carry the text through without parsing it. The Bouncer tests are covered by Card 9. No forced edit.
5. **Cards 4 and 7: new `round-<n>-snapshot.json` and `round-<n>-changes.md` files in the run directory.** Readers are `shedadapters/round.go` `parseRoundFileName`/`ReviewRounds`, which ignore unknown kinds, and `archiveRunDir`, which renames the whole directory. The other `round-` scans, `reportengine/archivedrecords.go:21` and `loomshed/prrework.go:19`, look at parent-review and rework directories. Nothing is forced.
6. **Card 12: the new `review_stalemate_rounds` key.** Readers are the template, `ConfigSpec`, `TestModules_EveryTemplateKeyClassified`, `TestConfigTemplate_ContainsEveryConfigYAMLTag` and `docs/overview.md:1443`. All covered by Card 12.
7. **Card 6: `prior_rounds` gets a new value.** Its only reader is `contracts/stencils/burler/burler-step-review.md:9,36`, which requires the marker non-empty; Card 6 keeps it non-empty. Covered.
8. **Card 8: the validate-review help text and doc lines.** `burlercli/doc.go`, `burlerengine/doc.go:560-561` and `docs/overview.md:1620` ("read-only review-gate self-check") are all covered by Card 8, apart from the stale key sentence in finding A.

**Findings**

- **A. LOW, consistency.** Location: Card 8 (`internal/burlercli#burlerCLI.validateReviewCmd`).
  - Summary: Card 5 makes `prior` a legal third top-level key, but the verb's `Long` help still says "The frontmatter's only top-level keys are verdict and findings". Card 8 lists the help text under Edit but names only the round-check and git-repository rewrites, so this sentence stays false.
  - Evidence: `internal/burlercli/validate.go:32-33`.
- **B. MEDIUM, scope.** Location: Card 10 (`internal/shedadapters/doc.go`, under Extends).
  - Summary: the package doc says a CONTINUE or CIRCLING round escalates "for one of three causes" and then "A fourth cause, decision". A `stalemate` escalation of an unconverged round makes that count false. The file is listed only to be added to, so no card rewrites the sentence. Move it under `**Edit:**` and rewrite the count.
  - Evidence: `internal/shedadapters/doc.go:140-142`.
- **C. LOW, scope.** Location: Card 12 (`docs/overview.md`).
  - Summary: the overview's paragraph on the review segment's escalations names circling, the hard cap and unreadable inputs as what the Bouncer escalates for. No card adds `stalemate` there; Card 12 edits only the review budget keys line.
  - Evidence: `docs/overview.md:1272-1284` and `docs/overview.md:1443`.

Method note: Card 10 lists doc comments to update but misses the package doc's prose count of causes and the overview's list of causes. Card 11 covers every other enumeration of the causes.

Tool calls:
1. Grepped production readers of fixer-reports, tests asserting the skipped-report text, and every "only keys" text: the manifest had no inventory of fixer-report consumers or review-key wording.
2. Grepped treadleengine's use of burlerengine and the cause-list texts across code and docs: I needed to know whether treadle parses burler reports, and where the causes are listed.
3. Grepped consumers of the cause constants outside shedadapters and arbiterstop, and cause lists in specs and stencils: these lie one hop past the manifest.
4. Grepped `shedadapters/doc.go` cause text, consumers of `ParentNotice`, and enumerations of round-file names: I needed doc text and directory scanners the manifest did not hold.
5. Read `shedadapters/doc.go:136-160` and grepped burlerengine's doc and specs for prior-round text: I needed the exact cause-count sentence.
6. Grepped `docs/overview.md` for escalation, fixer-report and validate-review text: Card 12 edits that file and the manifest had none of its text.
