```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change the diff makes, find what calls, reads, asserts or describes it, and report any member the diff leaves wrong.

**Per change: the set I found, and whether the diff covers it**

1. **`--drops-create` and the `Drops Create:` line.**
   - Covered: `loomcli/decision.go` (flag, refusals, Long help), `loomcli/doc.go`, `discussionparser/append.go` and `doc.go`, and `contracts/specs/loom-discussion-spec.md:56-58`.
   - Covered: the readers `runplan.PostBatchChecks`, plus the webster `postBatchInputs` and the weaver `record.go`/`resume.go` sites, which all pass `DecisionRecordPath`.
   - Not covered: the parent rule in `contracts/stencils/parent/parent-directive-parent.md:30-33` (finding A1).
   - Not covered: the `decision add` Long help drops rules that are still in force (finding A4).
2. **`createTargetWayForward` text and signature.**
   - Covered: its one caller, `failCardNotDone`, and the tests `TestFailCardNotDone_WayForward`, `TestRecordBatch_DoneChecksBlockOnUnresolvedCreate` and `TestRecoverBatch_Terminal…`.
   - Covered: `websterengine/doc.go:493-500`.
   - No stencil or spec quotes the old text, found by grep.
   - Not covered: `websterengine/doc.go:346` (finding A2).
3. **`verifyGateStuckFailure`: report path and drift route.**
   - Covered: its caller `verifyGateStuckReason`, the tests, and `doc.go:1210-1211`.
   - The `lym drift amend` verb exists and admits a blocked Webster (`driftcli/amend.go:36-37`), so the route it names is real.
   - Not covered: `doc.go:346` (finding A2).
4. **`BuildRoundContext` signature, now taking `RoundContextSources`.**
   - Covered: all its callers (`burlerreview.go:272`, `burlercli/validate.go:130`), the tests, and `shedadapters/doc.go`.
5. **`RoundContext.AmendingTexts` / `HaltReason`, `WebsterOrigin` and the round-check exemption.**
   - Covered: `burlerengine/doc.go`, the burler review-format and step-review stencils, and the prompt and roundcheck tests.
   - Not covered: the row wiring (finding A3).
6. **Halt-reason file `webster-halt.md`.**
   - Covered: the writer `WebsterProducer`, the readers (review producer, `validate-review`), the `runplan` doc constant and `shedadapters/doc.go`.
   - Its placement under the `.lym` conflict directory matches PATTERN-lym-files.
7. **`Env.AmendmentPath` / `Env.HaltConflictDir`.**
   - Covered: `loomcli/wiring.go` and `envkit.FullEnv`.
   - PATTERN-shed-recipe-registry is violated (finding A3).
8. **`validate-review` plan-dir recognition.**
   - Not covered: it hard-codes a recipe value (finding A5).
9. **`renderParentDecisions` numbering, and the master and implementer stencils.**
   - Covered: `RenderMasterPrompt`, `TestRenderPrompts` and `websterengine/doc.go:343-344`.
10. **Commit messages.**
    - The `Root-Cause:` trailers are present on the four fix commits.
    - No `Pattern-Departure:` or `Loosened-Check:` trailer is present. That matters for A3, and the removed `withNone` equality assertion in `template_test.go` is the tests lens's call.

**Findings**

- **A1 — MEDIUM, consistency.** Location: `contracts/stencils/parent/parent-directive-parent.md:33`.
  - The rule folds added entries into the main decisions with `decision replace`, then has the parent record each folded call again with a plain `lym loom decision add` before sending the run back to a Webster row.
  - `DroppedCreates` reads only inside added-decision entries (`discussionparser/append.go:239-275`), so a folded drop is lost.
  - A re-add without `--drops-create` brings the create-not-done block back.
  - The parent rule never tells the parent to re-pass `--drops-create`.
- **A2 — MEDIUM, consistency.** Location: `internal/websterengine/doc.go:346`.
  - The package doc still states "No refusal or prompt text names a loom verb."
  - The diff adds loom verbs to two Webster texts: `lym loom decision add` in `createTargetWayForward` (`recordbatch.go:117`) and `lym loom resume` in the drift route (`verifygate.go:611`).
  - Both texts can also reach a standalone Webster, which has no loom run.
  - Either the doc sentence is stale or the new texts break the package's own rule. The diff leaves the two disagreeing.
- **A3 — BLOCKING, design (PATTERN-shed-recipe-registry).** Location: `internal/shedrecipe/recipe.go:91-95` and `internal/shedrecipe/entries_burler.go:185`.
  - The diff adds two fields to `Env`: `AmendmentPath` ("read by the Plan-Burler-Review row alone") and `HaltConflictDir` ("Only the Plan-Burler-Review row reads it").
  - It also switches behaviour inside the shared `burlerReviewEntry` constructor on `row.Name == loomshed.NamePlanBurlerReview`.
  - The entry's background requires row values by key, "A row never gets its own `Env` field", and no switch inside the producer.
  - The plan-only behaviour should be a row config key read like `run_subdir`.
  - No `Pattern-Departure:` trailer covers this.
- **A4 — LOW, consistency.** Location: `internal/loomcli/decision.go:1984-2015`, the `decision add` Long help.
  - Commit e5b6b3a6 rewrote the help and dropped statements that are still true of the verb:
    - "It changes neither the support log nor the plan".
    - The `(--by operator only)` limit on the sampler session.
    - "is refused when the record declares none" for `--pattern-action withdraws`.
    - "no two share a title or a PATTERN entry" for `--from`; the refusal `decisionWayFromTwice` still exists at `decision.go:90`.
  - The help no longer documents those refusals, and nothing in the decision record asks for the removal.
- **A5 — LOW, consistency (PATTERN-code-conventions: named constants and no hard-coded choice).** Location: `internal/burlercli/validate.go:29-31,122`.
  - `planReviewRunSubdir = "plan"` restates a value the recipe owns: `contracts/recipes/tapestry-recipe.yaml:189,236,298` has `run_subdir: plan`.
  - The review producer instead keys the same behaviour on the row name (`entries_burler.go:185`).
  - If the recipe renames the subdir, `validate-review` silently stops applying the amendment and halt-reason exemptions, and so disagrees with the round-check gate.

**Tool calls**

1. Grep for the old way-forward text and the 400-byte wording in docs and specs, plus the `decision replace` fold and `ParseAddedDecisions` sites: the context did not show the text's readers or the fold path.
2. Grep the fold rule's stencil site and read `ParseAddedDecisions` and `committedLacksEntry`: needed to see whether a fold or re-add keeps `Drops Create:`.
3. Read the parent-directive fold lines and grep `drops-create` and `webster-halt` doc references: the stencil text was not in context.
4. Grep for an existing plan run-subdir constant and the `run_subdir` readers: needed to judge `planReviewRunSubdir` against the recipe.
