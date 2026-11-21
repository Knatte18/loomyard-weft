```yaml
lens: code-scope
outcome: reported
```

Task: map each change in the diff to the decision that asks for it, using the code-scope lens.

**Decisions (decision record alone; the corrupted Goal is replaced by the parent's Added after Discussion entry)**
- G1, Goal triage "Create check", which is Added part (1): the Create check reads the decision record, a dropped Create is reported as dropped-by-decision and does not block, and record-batch and recover-batch give the same way forward. Built by C1, C2 and C3.
- G2, Goal triage "Drift halt", which is Added part (2): the halt names a full report instead of cutting at 400 bytes, and the way forward names how the parent accepts expected drift. Built by C4.
- G3, Goal triage "Plan amendment", which is Added part (3): `--amends-plan` exempts the cards it names from the changes-file hunk rule, and Webster's halt reason reaches Plan-Burler-Fix as a finding. Built by C5 and C6.
- A1, Added after Discussion (parent, 2026-11-21): scope is parts (1) to (3) plus #178 part 1, a later added decision overrides an earlier one in the batch fork's prompt. The auto rebaseline after a fix (#170 bullet 2) is out of scope. #178 part 1 is built by C7. The exclusion holds: nothing in the diff runs a rebaseline after a plan fix. `createTargetWayForward` still tells the reader to run `lym webster rebaseline --card NN` by hand (`internal/websterengine/recordbatch.go:113`). `AutoRebaseline: true` in `WebsterProducer.Call` was already there before this diff.

**Changes and what they trace to**
- C1, the `--drops-create` flag that writes the `Drops Create:` line: `loomcli/decision.go` (flag, `dropsCreateRefusal`, `decisionInput.dropsCreate`), `discussionparser/append.go` (`AddedDecision.DroppedCreates`, `insertDecision`), the spec line in `contracts/specs/loom-discussion-spec.md`, both `doc.go` paragraphs and the tests. Traces to G1/A1 part (1).
- C2, the done-check skips a dropped target and warns: `runplan/postbatch.go` (`DecisionRecordPath`, `withoutDroppedCreates`, deferred warnings), `discussionparser.DroppedCreates`, `websterengine.postBatchInputs`, `runplan/doc.go` and the tests. Traces to G1.
- C2a, `weaverengine/record.go` and `resume.go` pass `DecisionRecordPath`. Forced by C2: weaver calls the same `PostBatchChecks`, and a Create the parent dropped should not block it either.
- C3, one way forward for both verbs: `createTargetWayForward` loses its `recovered` parameter and adds the drop route, with `failCardNotDone`, `websterengine/doc.go` and the tests updated. Traces to G1, "record-batch and recover-batch give the same way forward".
- C4, the verify-gate stuck reason names the report path and the drift entry gets the `lym drift amend` route: `verifygate.go`, `doc.go` and the test. Traces to G2.
- C5, amended cards are exempt from the round check: `burlerengine/roundcheck.go` (`AmendingTexts`, `namedByAmendment`), `prompt.go` (`amendsPlanMark` paragraph), `shedadapters/roundcontext.go` (`RoundContextSources`, `amendingEntryTexts`), `shedrecipe/entries_burler.go` (`amendmentTitlesAt`, Plan-Burler-Review only), `Env.AmendmentPath`, `loomcli/wiring.go`, the burler stencils and the tests. Traces to G3, first half.
- C6, the halt reason reaches plan review as an `origin: webster` finding: `runplan/conflict.go` (`HaltReasonName`, write, read and remove), `shedadapters/webster.go` (record on Stuck, clear on Done), `burlerreview.go` (read into `Profile.HaltReason`), `roundcontext.go` (`HaltReason`), `burlerengine` (`WebsterOrigin`, `haltReasonBlock`, the `newFindings` exemption), `Env.HaltConflictDir`, the wiring, the stencil lines and the tests. Traces to G3, second half.
- C5/C6a, `burlercli/validate.go` builds the amendment reader and conflict directory for the plan review run directory. Forced: the `BuildRoundContext` signature changed, and `validate-review` must agree with the review gate.
- C5/C6b, `testkit/envkit/envkit.go` fills `AmendmentPath` and `HaltConflictDir`. Forced: these are new `Env` fields, and `FullEnv` must fill every seam.
- C7, later decisions override earlier ones: `render.go` `renderParentDecisions` now numbers entries `Decision N:`, the master stencil and implementer body stencil gain supersede lines, and `doc.go` and `template_test.go` change with them. Traces to A1, #178 part 1.
- C8, the `lym loom decision add` help text is reworded and some sentences are dropped. Forced by C1: commit e5b6b3a6 greens the CLI-length test, and `cmd/lym/clitree_test.go:280` caps `Long` at `maxLongBytes`. Whether the dropped facts are still documented elsewhere belongs to the affected-set lens.

**Findings**

None from the code-scope lens. Every change traces to G1, G2, G3 or A1, or is an edit a traced change forces. Every decision is built by at least one change, and the excluded #170 bullet 2 is not implemented.

Outside this lens, noted only: C3 also offers `--drops-create` as a way forward for a rename-not-done finding. `withoutDroppedCreates` skips Create groups only (`internal/runplan/postbatch.go:144`), so that route cannot clear a Rename. This is for the correctness lens.

Tool calls:
1. `git show --stat e5b6b3a6` and a grep of `cmd/lym/clitree_test.go` for the `Long` length cap, to settle whether the help-text rewording in C8 was forced. My context held only the diff, not the commit's file list or the cap test.
