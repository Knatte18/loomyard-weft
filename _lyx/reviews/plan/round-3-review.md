---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/04-recorded-partition.md (Verbs) and _lyx/plan/07-rebaseline-tail.md (Intent)
    summary: "Card 4 edits the batch-forming line of internal/webstercli/rebaseline.go and card 7 replaces that same line with the new Active and Sizes deps, so card 4's edit survives only as a compile bridge"
  - id: F2
    severity: LOW
    class: design
    location: _lyx/plan/06-batcher-profiles.md (Tests, TestActive table)
    summary: "TestActive carries both an operator-only-profiles row naming cautious and a separate no-such-profile row, two rows for one error path through Active"
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/03-calibration-report.md (Run base from the transcripts, Tests)
    summary: "BaseSHA takes the earliest begin-batch envelope without saying earliest by what, and the test fixture is said to gain a webster session that the existing fixture and card 1 already hold"
  - id: F4
    severity: LOW
    class: scope
    location: _lyx/plan/07-rebaseline-tail.md (steps 5 and 6, Tests)
    summary: "No test row reaches the rebaseline tail's CheckBatchOrder refusal and its leave-state-unchanged bound, though Decision 12 puts the assertion at the tail re-batch"
---

## Focus answers

The four focus entries, each with its verdict.

**Webster call sites.**
Cards that edit each batch-forming line (`SequenceBatches(c.batcher.Batch(plan.Cards))`, or `deps.Batcher.Batch` plus `SequenceBatches` in `Run`):

| site | cards that edit it |
|---|---|
| `webstercli/beginbatch.go` | 4 |
| `webstercli/awaitbatch.go` | 4 |
| `webstercli/recordbatch.go` | 4 |
| `webstercli/recoverbatch.go` | 4 |
| `webstercli/validate.go` | 4 |
| `webstercli/rebaseline.go` | 4 and 7 |
| `websterengine/runlevel.go` `Run` | 4 |

Card 6 touches `webstercli/wiring.go` and test files, never a batch-forming line.
Only `rebaseline.go` is edited twice, and card 7 wholly overwrites card 4's edit: that is F1.

**Batcher signature change.**
Card 4 moves every production caller of `Batch` in the same card: `Run`, the six webstercli verbs and `identityBatcher`.
The in-package test implementers and callers (`emptyBatcher` and the direct call in `runlevel_test.go`, `identity_test.go`, `registry_test.go`'s `probeBatcher`) are all in card 4's Edit list.
`batcher_test.go` calls `Select` only, so it needs no change.
`internal/loomshed/webster.go` assigns the batcher and never calls `Batch`.
Two edits are discarded later: the `rebaseline.go` bridge (F1) and the `probeBatcher` signature in `registry_test.go`, which card 6 deletes.
The second is not a finding: the probe has to satisfy `Batcher` for as long as the registry it tests exists, and nothing short of moving card 6's deletion into card 4 avoids it.

**`RunTally.BaseSHA`.**
Card 3 names the test (`TestCountRun`'s fixture) and states the recognition rule: a `Bash` `tool_use` whose command invokes `lyx webster begin-batch`, its `tool_result` paired by `tool_use_id`, whose content parses as a JSON object with a non-empty `start_sha`.
The rule matches `webstercli/beginbatch.go`, whose envelope carries `start_sha` at the top level beside `ok`, and the fixture covers an error envelope, an unrelated result mentioning `start_sha`, and a run with no pair.
The gap is the ordering and the fixture wording: F3.

**`internal/batcher` tests under `PATTERN-test-economy`.**
Card 2's `ProfileWeights` rows (absent profile, missing, negative and unknown coefficient, valid profile) and card 6's `Active` rows reach different public surfaces, and card 6 says so for the coefficient cases.
Card 5's `cost_test.go` rows each pin a distinct property.
The one overlap is inside card 6's `TestActive`: F2.

## Findings

### [MEDIUM] Card 4's `rebaseline.go` edit is wholly overwritten by card 7

**Location:** `_lyx/plan/04-recorded-partition.md` (Verbs, last paragraph) and `_lyx/plan/07-rebaseline-tail.md` (Intent).
**Issue:** Card 4 changes the `Batch` signature, so `internal/webstercli/rebaseline.go` must change to compile, and card 4 rewrites its batch-forming line to take the active batcher's output over `DiskSizes` and refuse on `CheckBatchOrder`.
Card 7 then replaces the `Batches` dep with `Active` and `Sizes` and deletes that line.
Card 4 says so itself: "the one call site the next card edits again".
The edit exists only to keep the tree compiling between the two cards, which is the shape the sequencing rule exists to avoid.
**Fix:** Move the dependency-shape change into card 4.
In card 4, `RebaselineDeps` gains `Active batcher.Batcher` and `Sizes batcher.SizeSource` in place of `Batches`, `websterengine.Rebaseline` forms the batches from them with the new signature and runs `CheckBatchOrder`, and `webstercli/rebaseline.go` passes `c.batcher` and `batcher.DiskSizes(c.geom.WorktreeRoot)`; card 4's Edit list gains `internal/websterengine/rebaseline.go`, `rebaseline_test.go` and `rebaseline_git_test.go` for the deps rename.
Card 7 then edits `websterengine/rebaseline.go` and its tests for the kept prefix and tail re-batch, and `webstercli/rebaseline.go` for its help text only, so no edit of card 4's is discarded.
Card 7's sentence "`RebaselineDeps` replaces `Batches` with ..." moves to card 4.

### [LOW] `TestActive` has two rows for one error path

**Location:** `_lyx/plan/06-batcher-profiles.md`, Tests.
**Issue:** One row has a file whose `profiles:` holds only an operator profile and whose `active:` names `cautious`, failing to load; a separate error row is "no such profile".
Both end in `Active` returning the `active:` names no profile error.
The first row already proves the second, and additionally pins that `Active` passes `ConfigOpenMaps()` so the template's `cautious` is not filled in, so the second adds no behavior.
**Fix:** Make the operator-only-profiles row the no-such-profile row, asserting the error names `batcher.yaml` and the profile, and drop the separate row.

### [LOW] `BaseSHA` ordering is unstated and the fixture wording disagrees with card 1

**Location:** `_lyx/plan/03-calibration-report.md`, "Run base from the transcripts" and Tests.
**Issue:** `CountRun` visits session files sorted by file name, which is not time order, and a Webster run that pauses or resumes can leave more than one `webster` session file.
"The earliest ... envelope" does not say whether earliest is by file order or by the line's `timestamp`, and the fixture's two pairs sit in one file, so the rule is not pinned.
The Tests paragraph also says the fixture "gains a `webster` session"; the existing fixture's `b.jsonl` is already the `webster` session, and card 1 already gives it sub-agent transcripts.
**Fix:** State that earliest means the smallest `timestamp` on the envelope's `tool_result` line, with `ForkTally.Started`'s timestamp reading reused.
Say the existing `webster` session gains the pairs, and put the two successful pairs in two `webster` session files whose name order is the reverse of their timestamp order, inside the one `TestCountRun` fixture.

### [LOW] No test reaches the rebaseline tail's order refusal

**Location:** `_lyx/plan/07-rebaseline-tail.md`, steps 5 and 6 and Tests.
**Issue:** Step 5 runs `CheckBatchOrder` over the kept batches and the tail, and step 6 replaces `State.Partition` only after every check passes.
The Tests list pins "leaving `Partition` unchanged" only for the card-set-changed refusal.
Decision 12 puts the assertion at a rebaseline's tail re-batch, and `PATTERN-refusal-way-forward` requires a test to reach each refusal site; a tail with a backward dependency would currently refuse untested, and nothing pins that the refusal leaves `State` unchanged.
**Fix:** Add one row to the existing refusal test, not a new function: a plan whose unbegun tail holds a card that uses a later card's target is refused with `ErrBatchOrder` and leaves `State`, `Partition` included, unchanged.
