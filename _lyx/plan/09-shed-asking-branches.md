# Card 9 — shed-asking-branches

**Edit:**
- `internal/shedadapters/singlellm.go`
- `internal/shedadapters/burler.go`
- `internal/shedadapters/webster.go`
- `internal/shedadapters/doc.go`
- `internal/shedadapters/singlellm_test.go`
- `internal/shedadapters/burler_test.go`
- `internal/shedadapters/webster_test.go`
- `internal/shedadapters/bouncer_judge_test.go`
- `internal/shedadapters/bouncer_seed_test.go`
- `internal/loomshed/discussionwrite.go`
- `internal/loomshed/planwrite.go`
- `internal/loomshed/discussionwrite_test.go`
- `internal/loomshed/planwrite_test.go`
- `internal/shedrecipe/entries_discussionwrite_test.go`
- `internal/shedrecipe/entries_planwrite_test.go`
- `contracts/recipes/loom-recipe.yaml`

**Intent:**
D7 for the shed producers: every branch that existed only to handle a shuttle asking result goes, now that `Wait` cannot return one.

- `SingleLLMProducer.mapOutcome` loses its `OutcomeAsking` case and the "agent is asking a question" Stuck; an unexpected outcome still reaches the existing default branch.
  Rewrite the probe comment in `Call` that names `spec.AwaitOperator`.
- `BurlerProducer` loses the asking case in its attempt loop and in `probeLiveRound`.
- `WebsterProducer.Call` loses its `websterengine.ErrMasterAsking` branch, and `masterAskingReason` is deleted.
  This removes the last reference to `ErrMasterAsking` and `MasterAskingError` outside `websterengine`, which card 10 deletes.
- `shedadapters/doc.go` drops the asking rows of its outcome mapping;
  `loomshed/discussionwrite.go` and `loomshed/planwrite.go` drop the sentences that name the asking Stuck path;
  the `loom-recipe.yaml` header comment drops the asking path from Discussion-Write's reachable Stucks, keeping the gate-exhaustion one.
- No refusal or spec row names these asking outcomes (`contracts/specs/refusal-spec.md` has none for them); confirm with a grep before committing.

Tests: delete the tests whose only subject is the asking mapping (`TestPlanWriteEntry_CallAsking`, `TestDiscussionWriteEntry_CallAsking`, and the asking rows or cases in `singlellm_test.go`, `burler_test.go`, `webster_test.go`).
Where a test uses `OutcomeAsking` only as some non-done outcome (`bouncer_judge_test.go`, `bouncer_seed_test.go`, the attach row in `entries_discussionwrite_test.go`), switch it to `OutcomeDied` or `OutcomeTimeout`, keeping what it asserts.
Update the comments in `loomshed`'s two tests that describe the asking path.

**ImpactSummary:** Shed producers no longer map an asking result; Discussion-Write and Plan-Write Stuck only on an exhausted gate, and Webster no longer Stucks on an asking Master.
