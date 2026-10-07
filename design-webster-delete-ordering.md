# A valid plan runs in card order: Plan-Gate refuses backward dependencies, Webster never reorders

`webster-delete-ordering` · Note · bug, webster

## Rule (operator, 2026-10-06)

A valid plan is executable strictly in card-number order: implementing card 1, then 2, then 3 and so on always works.
A card never depends on anything a later card does.
Webster follows that order as written and never sorts cards.

This reverses `contracts/specs/loom-plan-spec.md`, which today calls the card list a DAG of intent "not itself an execution order" and lets the executor derive the order (`websterengine.SequenceBatches`, `deriveEdges` in `internal/websterengine/sequence.go`).

## What changes

- **The plan spec and the plan template** state the rule to the plan writer and the plan reviewer.

## Sightings

- 2026-10-03, `pair-lifecycle` (GitHub #350): card 12 deleted `fabricengine.Topology.RemovePairBranch`, `PairBranchResult` and `PairSiblingRemnant` and ran before card 11, which removed their only production caller in `internal/battencli/wire.go`; the batch failed and the run blocked at Webster.
- 2026-10-06, `skills-in-one-turn`: card 2's `Uses:` named `internal/shuttleengine#Runner.skillLoader`, which card 4 edits, a backward dependency; cards 2 and 4 also shared test-file targets.
  `SequenceBatches` condensed the cycle {2, 4} and ran 01, 02, 04, 03, so card 4 deleted `SkillLoader.SkillLoadSequence`, `SkillUnknown` and `Runner.LoadSkill` before card 3 moved `internal/orchengine` and `internal/orchcli` off them.
  The cycle surfaced only as an unread `Cycle` warning.
  `webster rebaseline` refused the fix because batch 4 had begun, so the run needed `webster reset --to start`, an archived `state.json` and a goto, redoing two finished batches.

Plan-Gate refuses backward dependencies since `bugfix-webster` (d536a1a4e: `uses-later-target`, `delete-before-reference`).
Webster dropping its own ordering moved to task `card-batching` (its Decision 12, 2026-10-07): the batchifier's contiguous segments in plan card order are the execution order, and `SequenceBatches` reduces to an assertion that refuses a backward edge or cycle.
This note keeps stating the rule in the plan spec and template.
Until then Plan-Write can bounce on plans Webster used to reorder, since the template does not yet tell the writer.
