```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens, which traces each change in the diff to the amended board entry's goal points.

**Decisions.** These are the goal points of the board entry under `## Goal`. The record has no `Added after Discussion` entries.
- G1: once any batch has begun, the plan-fix stage refuses or warns on edits to `00-overview.md` outside its Card Index.
- G2: glyph validation accepts a target that an earlier card of the plan declares as created.
- G3: a `glyph-not-found` finding prints the unit's actual member names.
- G4: a finding says whether the member is expected to be created by a card.
- G5: when a recovery ends with `dead_reason: asking`, the agent's last message goes into the halt or refusal note.
- G6: either the question goes to the parent, or a plan rebaseline that touches the dead batch's cards re-arms one recovery, so no reset is needed.

**Changes and what they trace to.**
- C1. `internal/loomshed/gates.go` `overviewFrameEditedFindings` falls back to the baseline-copy frame, and `unframedOverviewFindings` refuses any overview edit when there is no frame. `TestNewPlanGate` gets new rows. Traces to G1.
- C2. One line in `contracts/stencils/loom/loom-rubric-plan-review.md` says the overview outside its Card Index is frozen. Traces to G1.
- C3. `runplan.RecordedOverviewFrame` moves out of websterengine, and `rebaseline.go` and `beginbatch.go` call the moved copy. Forced by C1: the gate shares the rebaseline's reader.
- C4. The `overview-frame-edited` text in `contracts/specs/loom-plan-checks-spec.md` is updated. Forced by C1.
- C5. `internal/planglyph` adds `createdHandleGlyphs`, `createdBy` and the creator branch in `statusFindings`. The `DetectDrift` and `resolve_test.go` call sites get the new arguments. Traces to G2 and G4.
- C6. `siblingMembers`, `missingMemberSiblings` and `missingDetail`'s member list, with `TestValidateFormat_MissingMemberFindings`. Traces to G3.
- C7. `planparser.goPackageDir` is exported as `GoPackageDir` and its references renamed. Forced by C6: `siblingMembers` lists from an external-test unit's package directory.
- C8. The `status_enforcement_test.go` allowlist entries. Forced by C5 and C6.
- C9. The glyph text in the planglyph `doc.go` and the plan-checks spec. Forced by C5 and C6.
- C10. `gatefixer.LastTurnMessage` and its test, the `awaitTerminal` asking-to-waiting branch, `askingNeed`, the new `TestRecoverBatch_SecondCall` row and the `waiting.go` header. Traces to G5 (the message becomes the `need`) and to G6's route to the parent.
- C11. `Rebaseline` records `AmendedCards` for terminal batches, and `recoveryRetryRead` lets an unrendered amendment pass the cap. This brings the `TestRecoveryRetry` rows, the rebaseline rows using `checkTerminalCardAmended`, and the changed `beginbatch_test.go` row. Traces to G6's re-arm route.
- C12. The Master template's waiting and `recovery_retry` rungs, the `recover-batch` `Long` help and the websterengine `doc.go`. Forced by C10 and C11.

**Decisions and the changes that build them.**
- G1: C1 and C2.
- G2: C5.
- G3: C6.
- G4: C5, through `missingDetail`'s "no card of the plan creates it" and "is created by card N".
- G5: C10.
- G6: C10 and C11. The entry offers "either"; the diff takes both routes, and the summary records that choice. Taking both stays within what the entry permits, so I raise no finding.

No decision is left unbuilt.

**Findings**

1. LOW, scope: the re-arm reaches failed and stuck batches as well as dead ones.
   - Location: `internal/websterengine/rebaseline.go:371-378`, which records `AmendedCards` for any terminal batch whose edit was accepted.
   - The entry asks that a rebaseline touching the dead batch's cards re-arm one recovery. The diff also records an unrendered amendment on terminal failed and stuck batches.
   - So those batches get an uncounted recovery spawn: `recoverbatch.go:385` skips the count when `rendersNewAmendment(prior)`.
   - They also skip the cap at `recoverbatch.go:520`, `runlevel.go:1191`, `beginbatch.go:226` and `recordbatch.go:177`. The entry does not license this for failed or stuck batches.
   - Evidence:
     - `internal/websterengine/rebaseline.go:220-224` and `:371-378`
     - `internal/websterengine/doc.go:106-109`, which says "A named card of such a batch [failed, dead or stuck] … is also recorded in AmendedCards"
     - the decision record's Goal, lines 73-74: "let a plan rebaseline that touches the dead batch's cards re-arm one recovery"

No unrequested change beyond finding 1, and no decision left unbuilt.

Tool calls: none.
