```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change the diff makes, list what reads, asserts or describes it, and flag anything the diff leaves wrong.

**Per change, the set I found, and whether it is covered**

1. **`RoundContext.FocusFindings` renamed to `HardenerRound`, and the hardener exemption widened** (`roundcheck.go`). Covered.
   - Writers: `burlercli/validate.go:137` and `shedadapters/roundcontext.go` (`RoundContextSources`, `BuildRoundContext`). Both updated.
   - Producer path: `shedadapters/burlerreview.go` (`BurlerHalfDeps`, the producer field, `roundProfile`) and `shedrecipe/entries_burler.go:183`. Both updated.
   - Docs: `burlerengine/doc.go:308-312`, the `CheckReviewRound` doc and `validate.go`'s long doc (the "BLOCKING bound of an incremental round" at line 49 is still true for every other segment). All correct.
   - Tests: `roundcheck_test.go`, `roundcontext_test.go`, `entries_burlerhalves_test.go` and `validate_integration_test.go:96-97,111`. The integration test's `FocusFinding…` rows still hold under the wider rule, because a BLOCKING decision finding outside the changes file is still admitted in the hardener directory and refused in the code directory.
   - Go-rendered prompt: `burlerengine/prompt.go` `priorFindingsBlock` (lines 488-545) states only "This round is incremental." and names no BLOCKING or changed-file rule, so nothing there contradicts the exemption.

2. **`FocusOrigin` constant removed.** Covered. A grep shows no Go reference left. The `origin: focus` string still appears in these places, and each is consistent with the rubric, which keeps `origin: focus` for an undriven or WARN entry only:
   - `burler-review-format.md:59`
   - `burler-cluster-fan.md:32`
   - `burler-assist-cluster-fan.md:29`
   - `burler-assist-consolidation.md:11`
   - `harden-rubric-review.md:54`
   - test fixtures in `validate_integration_test.go:111` and `entries_burlerhalves_test.go:331`

   Parsing doesn't break: origin is a free string, and only `ParentOrigin` and `WebsterOrigin` are matched (`parentseat.go:17,21`, `roundcheck.go:305,319,398`).

3. **Review-format and review-step stencil wording for the hardener round.** Covered.
   - `burler-review-format.md:78-79` and `burler-step-review.md:55-56` agree with Go: new findings at any severity and in any file, carried findings to any file, class and severity carry rules kept.
   - `burler-step-review.md`'s list structure still reads correctly: the new bullet is a top-level sibling after the incremental bullet.
   - The copy of `burler-step-review.md` rendered into this round's own instruction 2 (line 107) still has the old wording. That is the deployed board copy from before the merge, not the diff, so it is no finding.

4. **`decisionRuleMarker` gains `firstRaisedGates`, set by a `harden_round` Bouncer through `BouncerConfig.FirstRaisedGates`.**
   - Callers: `bouncer.go:2083` and `bouncer_seed_test.go:488`. Both updated.
   - Docs: `bouncerprompt.go`'s doc and `shedadapters/doc.go:160-162`. Both updated.
   - Judge stencil `bouncer-template-judge.md`:
     - Line 44, "where the decision rule says so", still holds.
     - Line 53 still holds.
     - Line 13, its maintainer comment, is **not covered**: it still says the marker renders the lighter rule from round 2 on (finding F1).
   - Go paths outside the prompt: `settleUndecided`, `softCapRule` and `designWrongKeys` (`bouncerfiles.go:259-271`) never look at first-raised status, so they need no change.
   - `hardenshed/doc.go` and `docs/overview.md` state no convergence or first-raised rule, so they are unaffected.

5. **Rubric rules: FAIL, kill both ways, whole repro, class rule, Converged, follow-ups** (`harden-rubric-review.md`).
   - The other stencils that describe origin and class (item 2) still agree.
   - The judge template reads the rubric as a marker value, and the Converged bullet agrees with the rendered rule when `FirstRaisedGates` is set.
   - Harden seed, `hardenshed` doc and overview: a grep finds no restated rule.

6. **The fix row's neighbour instruction** (`hardener-recipe.yaml:149-151`). It is a free-text recipe value, and nothing reads or asserts it.

7. **Commit messages and declared PATTERN changes.**
   - No commit in the range carries a `Pattern-Departure:`, `Loosened-Check:` or `Root-Cause:` trailer.
   - The directive declares no PATTERN changes, so there is no landing to check.
   - The commit messages agree with the diff.
   - `roundcheck_test.go` loses refusal rows with no `Loosened-Check:` trailer. The Loosened checks bullet owns that, and it falls outside this lens.

**Findings**

- **F1.** Severity LOW, class `consistency`.
  - Location: `contracts/stencils/bouncer/bouncer-template-judge.md:13`.
  - Summary: the stencil's maintainer comment says `{{.decision_rule}}` "renders round 1's rule in round 1 and the lighter rule from round 2 on". A hardener Bouncer with `FirstRaisedGates` now renders round 1's rule in every round (`internal/shedadapters/bouncerprompt.go:28`, `internal/shedrecipe/entries_bouncer.go:372`), so the comment misdescribes the variant.
  - Evidence: `bouncer-template-judge.md:13`, `bouncerprompt.go:24-29`.
  - Fix: add "unless the Bouncer sets FirstRaisedGates, which renders round 1's rule in every round" to that comment.

Tool calls:
1. A grep of the harden and bouncer stencils, `internal/hardenshed`, `docs/overview.md` and the hardener recipe for restated convergence or round rules. The manifest had not swept those files for stale wording.
2. A grep of `internal/burlerengine/prompt.go` for incremental, changes-file or focus text, to find Go-rendered rule wording that the manifest had not covered.
3. A read of `internal/burlerengine/prompt.go:486-560` (`priorFindingsBlock`), to confirm that no Go-rendered text states the BLOCKING or changed-file rule.
