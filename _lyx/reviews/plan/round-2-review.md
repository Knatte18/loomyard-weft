---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/04-batcher-takes-sizes.md
    summary: Card 4 rewrites the batch-forming line in six webstercli verbs and `Run`, and card 7 rewrites the same lines again, with card 8 replacing the rebaseline one a third time, so card 4's caller edits have no surviving reviewable content
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/03-calibration-report.md
    summary: Card 3 adds `RunTally.BaseSHA` extraction from the earliest `begin-batch` result envelope but plans no test for it, and does not say how an envelope is told from other tool results
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/06-batcher-profiles.md
    summary: The `TestActive` row asserting a negative coefficient through `Active` re-tests the parse error that card 2's `ProfileWeights` rows already pin and that the new `verbs_test.go` rebaseline step already reaches through `Active`
---

### [MEDIUM] Card 4's caller edits are overwritten by card 7 and card 8

**Location:** `_lyx/plan/04-batcher-takes-sizes.md` (`webstercli/beginbatch.go`, `awaitbatch.go`, `recordbatch.go`, `recoverbatch.go`, `rebaseline.go`, `validate.go`, `websterengine/runlevel.go`, `runlevel_test.go`) against `_lyx/plan/07-recorded-partition.md` and `_lyx/plan/08-rebaseline-tail.md`.

**Issue:**
Today each of those verbs holds one batch-forming line, `websterengine.SequenceBatches(c.batcher.Batch(plan.Cards))`, and `Run` holds the equivalent at `runlevel.go:409`.
Card 4 edits each line to pass the plan and `batcher.DiskSizes(...)` and to refuse a `Batch` error as raw I/O.
Card 7 then replaces the same line in every one of them with `ExecutionBatches(...)` over the recorded partition, and moves `Run`'s computation to after the state phase.
`webstercli/rebaseline.go` is rewritten a third time by card 8, which replaces `RebaselineDeps.Batches` with `Active` and `Sizes`.
After card 7, the only trace of card 4 in these files is the signature of a call that card 7 no longer makes.
Card 7's own `Intent` justifies being one card because "every verb that forms batches changes its call once", and card 4 contradicts that by making every one of them change twice.
Card 4's surviving content is the `internal/batcher` package (`Batch.Profile`, `Batch.Estimate`, `Identity()`, the new `Batch` signature), which cannot compile without its callers, so it cannot be split off from them.
`registry_test.go`'s probe batcher is also edited by card 4 and deleted by card 6; that one-line signature edit is forced for compilation and is not part of this finding.

**Fix:**
Fold card 4 into the recorded-partition card, so the `Batcher` signature change and every caller's move to `ExecutionBatches` land in one commit, and put the merged card straight after card 3, before the cost batchifier.
The merged card does not depend on cards 5 or 6: it needs only `Identity()` and the active batcher's `Batch`.
Renumber the rest so numbering stays gapless: the cost batchifier stays 5, profiles stay 6, rebaseline-tail becomes 7 and calibrate-template becomes 8; update the Card Index, the `Shared Decisions` card 9 reference and every "card N" cross-reference in the cards.
`webstercli/rebaseline.go` keeps one transitional edit in the merged card (the active batcher's output under `CheckBatchOrder`), which card 7 replaces; it is the only call site where the order of the two cards forces a second edit, and the merged card says so.

### [MEDIUM] Card 3 has no test for the `BaseSHA` extraction

**Location:** `_lyx/plan/03-calibration-report.md`, section "Run base from the transcripts" and "Tests".

**Issue:**
The card adds a behavior to `CountRun` in `count.go`, recording `RunTally.BaseSHA` from the earliest `lyx webster begin-batch` result envelope in the run's `webster` session transcripts.
Its tests are `calibrate_test.go`, which drives the calibration over an in-memory history and hand-built run tallies, and `calibrate_integration_test.go`, which builds a repository.
Neither reads a transcript, and `count_test.go`, the file whose `TestCountRun` fixture covers `CountRun`, is not among the card's targets.
So the one step that turns transcripts into the base tree every estimate depends on is untested, and the card's own coverage rule (the card's targets include the test that covers the behavior it introduces) is not met.
The card also does not say how an envelope is recognised.
In a real Master transcript the `begin-batch` result is the string content of a `tool_result` whose paired `tool_use` is a `Bash` call with the command `lyx webster begin-batch <N>`; the envelope carries `start_sha`, and a refused call returns an error envelope without it.
A card that leaves the rule open lets an implementer match any tool result that mentions `start_sha`.

**Fix:**
Add `tools/tokencount/count_test.go` to card 3's `**Edit:**` targets and extend `TestCountRun`'s fixture with a `webster` session holding two `begin-batch` tool-use and tool-result pairs, the earlier one carrying the expected `start_sha`, plus an error envelope without `start_sha` placed first, and assert `BaseSHA` equals the earliest successful one; a run with no such pair keeps `BaseSHA` empty.
State the recognition rule in the card's "Run base from the transcripts" paragraph: the `tool_result` paired by `tool_use_id` with a `Bash` `tool_use` whose command invokes `lyx webster begin-batch`, whose content parses as a JSON envelope with a non-empty `start_sha`.

### [LOW] `TestActive`'s negative-coefficient row duplicates two other tests

**Location:** `_lyx/plan/06-batcher-profiles.md`, section "Tests".

**Issue:**
Card 2's `config_test.go` rows pin that `ProfileWeights` rejects a missing, negative and unknown coefficient.
Card 6's `verbs_test.go` step runs `rebaseline` under a `cost` profile with a negative coefficient and asserts exit 1 and a message naming `batcher.yaml`, which reaches the same error through `Active` and the prologue.
`TestActive` then adds "one row where a negative coefficient shows card 2's parse error naming `batcher.yaml` through `Active`".
That row's only extra claim, that `Active` returns `ProfileWeights`' error rather than swallowing it, is already asserted by the `verbs_test.go` step, so the row covers nothing the other two do not (`PATTERN-test-economy`).

**Fix:**
Drop that row from `TestActive` and say in the card that the `verbs_test.go` step pins the pass-through; keep the other `TestActive` rows, which each cover an `Active`-only load error.

## Focus verdicts

1. **Webster call sites.**
   `webstercli`'s six verbs and `Run` are edited by both card 4 and card 7 in a way the later card overwrites; `rebaseline.go` is rewritten by cards 4, 7 and 8. Filed as F1.
2. **Config open maps.**
   Not a defect.
   Card 6 declares `profiles` an open map in `config.go` (`ConfigOpenMaps()`), sets it on the `batcher` entry in `configreg.Modules()`, and passes it through the shared load helper to `configengine.LoadOrTemplate`.
   `yamlengine.FillMissing` skips a present key at an open-map path and appends the template's value only when the key is absent, which is what the card states, and `Reconcile` carries the operator's value whole.
   The ordering is also sound: between card 2 and card 6 the template holds no `profiles:` key, so the load neither fills nor prunes anything.
3. **Fork attribution in `tools/tokencount`.**
   Not a defect.
   I checked 911 real sub-agent transcripts under `~/.claude/projects`: the prompt file is read by the `Read` tool as a string result with an unpadded `<number><TAB>` prefix (`58<TAB>- ...`), exactly one tool result per fork holds pointer-shaped lines, the gate lines of that same result carry `: <command>` after the closing backtick, and a fork's first timestamped line is its own, since inherited context is a `fork-context-ref` line rather than copied messages.
   The card's rule (strip the prefix, bullet with nothing after the closing backtick, first pointer-holding result only, string or list content) and its fixture work against that form.
4. **Test economy across the batcher cards.**
   One duplicate: the `TestActive` negative-coefficient row. Filed as F3.
   Card 2's `ProfileWeights` absent-profile row and card 6's `active:` names no profile row cover different surfaces (the calibration's profile lookup against `Active`'s resolution), and the remaining `TestActive` rows each pin an `Active`-only error.

## Checked and found sound

- Decisions 1 to 14 and every Constraint are carried: the card order matches Decision 1, the cost formula and coefficient keys of card 2 match Decisions 4 and 5, card 5 matches Decisions 6 and 7, card 6 matches Decision 8, the recorded partition, order assertion and tail re-batch of cards 7 and 8 match Decisions 10 to 12, and the spec, doc and `PATTERN-batcher-registry` edits land with their cards.
- Every package a card targets has its tests in `verify:`, including the `integration` tests of `gitrepo`, `websterengine`, `webstercli`, `loomshed` and `tools/tokencount`; `internal/batcher`, `internal/planparser` and `internal/configreg` carry no build-tagged tests.
- Every file card 6 must change for the removal of `batcher.Select` is in its targets (`cli_test.go`, `verbs_test.go`, `runlevel_test.go`).
- Each new refusal (batch order, recorded partition mismatch, `batcher.yaml` did not load) lands its spec row in the card that introduces it and is reached by a test.
- Cards that add a verb, flag or routing edge state what it can let through and its bound.
