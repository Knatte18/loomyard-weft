---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/07-batch-order-assertion.md
    summary: Cards 4, 7 and 8 rewrite the same webster call sites three times, and card 8 overwrites every call-site edit card 7 makes; card 7 belongs inside card 8
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/06-batcher-profiles.md
    summary: profiles is a map whose keys the operator owns but no card declares it an open map in configreg, so lyx config reconcile prunes operator profiles and the load re-injects the template's identity and cautious profiles
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/plan/01-tokencount-fork-report.md
    summary: The attribution rule does not match the real fork transcript, where the card pointer lines sit in a Read tool result behind a cat -n line-number prefix and the card gate lines share their leading shape, so a literal implementation attributes no fork
  - id: F4
    severity: LOW
    class: design
    location: _lyx/plan/06-batcher-profiles.md
    summary: Card 6 re-tests the missing, negative and unknown coefficient errors through Active, behavior card 2 already covers through ProfileWeights on the same parse
  - id: F5
    severity: NIT
    class: consistency
    location: _lyx/plan/10-calibrate-template.md
    summary: Card 10 says the worktree batcher.yaml resolves to the template when absent, but the file is present and holds only the active key, so the profiles come from the key fill
---

### [MEDIUM] Cards 4, 7 and 8 edit the same call sites three times

**Location:** `_lyx/plan/04-batcher-takes-sizes.md`, `_lyx/plan/07-batch-order-assertion.md`, `_lyx/plan/08-recorded-partition.md`

**Issue:**
Every webstercli verb that forms batches today (`beginbatch.go`, `awaitbatch.go`, `recordbatch.go`, `recoverbatch.go`, `validate.go`) holds one line, `websterengine.SequenceBatches(c.batcher.Batch(plan.Cards))`, and `Run` holds the same pair at `runlevel.go:409` and `:426`.
Card 4 rewrites that line for the new `Batch` signature.
Card 7 rewrites it again to `Batch` plus `CheckBatchOrder`, because removing `SequenceBatches` forces every caller to change.
Card 8 rewrites it a third time to `ExecutionBatches`, which already runs `CheckBatchOrder` itself, so the call-site edits of card 7 survive nowhere in the five verb files and `Run`.
The same holds for `runlevel_test.go`, which cards 4, 7 and 8 all edit.
Card 7 is not an independently reviewable unit: its only reviewable content that survives is `CheckBatchOrder`, the removal of `SequenceBatches` and its docs, and card 8 is where the verbs' final shape is decided.
`webstercli/rebaseline.go` is also edited by cards 4, 7 and 9, but there each edit is forced by compilation (card 9 replaces the `Batches` dep), so it stays.

**Fix:**
Merge card 7 into card 8 as one card, "recorded partition and batch order", and renumber the later cards.
The merged card creates `CheckBatchOrder` and `ExecutionBatches`, removes `SequenceBatches`, the cycle machinery and the `cycles` key, records the partition, and edits each verb file once.
`rebaseline.go` in webstercli keeps computing from the batcher plus `CheckBatchOrder` until the rebaseline card replaces it.

### [MEDIUM] `profiles:` is not declared an open map

**Location:** `_lyx/plan/06-batcher-profiles.md` (and `_lyx/plan/02-batcher-estimator.md`, which adds the shared load)

**Issue:**
`profiles:` is keyed by names the operator chooses, so it is an open map in the sense of `configreg.Module.OpenMaps` and `yamlengine`.
No card touches `internal/configreg/configreg.go`, and card 2's shared load helper and `Active` pass no open maps to `configengine.LoadOrTemplate`.
Without the declaration, `lyx config reconcile` (`configsync`) prunes every profile the operator adds that the template lacks, and `yamlengine.FillMissing` re-injects the template's `identity` and `cautious` profiles, and any missing coefficient of a profile named `cautious`, into the in-memory config on every load.
That defeats the Decision 8 load errors for those names (an `active:` naming a profile the operator deleted still resolves, a missing coefficient never errors) and makes named profiles non-durable.

**Fix:**
`internal/batcher` gains `ConfigOpenMaps()` returning `profiles`.
The shared load helper passes it, `configreg.Modules()` sets it on the `batcher` entry, and `configreg_test.go`'s `TestLookup` gains a `batcher` row.
`TestActive` gains one row showing a file whose `profiles:` lacks `cautious` and whose `active:` names it fails to load, which only holds when the template's profiles are not filled in.

### [MEDIUM] Card 1's attribution rule does not match the real fork transcript

**Location:** `_lyx/plan/01-tokencount-fork-report.md`

**Issue:**
In real Webster fork transcripts (read under `~/.claude/projects/<encoded bugfix-webster>/<session>/subagents/`), the fork reads its prompt file with the `Read` tool, and the pointer lines arrive in that tool result in `cat -n` form, for example `58<TAB>- \`_lyx/plan/01-uses-later-target.md\``.
The same prompt also holds card gate lines of the shape `70<TAB>- \`_lyx/plan/01-uses-later-target.md\`: \`go build ...\``.
Card 1 describes the pointer as a bare line `- \`<path>/NN-<slug>.md\``, says nothing about the line-number prefix, and does not separate a pointer line from a gate line.
An implementation and fixture written to the card's literal text finds no pointer in a real transcript, so every fork is listed unattributed and the calibration (card 3) skips every card, and card 10 can never see a good fit.

**Fix:**
State in the card that the fork's prompt is the `Read` result of its prompt file, that each line carries a `<number><TAB>` prefix to ignore, and that a pointer line is one that ends after the closing backtick, which excludes the gate lines.
The `count_test.go` fixture builds the tool result in that `cat -n` form, with a gate line naming a third card that must not count.

### [LOW] Card 6 repeats card 2's coefficient-error rows

**Location:** `_lyx/plan/06-batcher-profiles.md`, **Tests** paragraph

**Issue:**
Card 2 plans `config_test.go` rows for a missing, negative and unknown coefficient and a missing profile, all asserting `ProfileWeights`' errors.
Card 6's `TestActive` plan lists an error row for each load error "and card 2's missing, negative or unknown coefficient", which runs the same parse through `Active`.
That is the same behavior covered twice; `PATTERN-test-economy` asks for a row only where a behavior is not yet covered.

**Fix:**
Card 6's `TestActive` rows cover what `Active` adds: `active:` names no profile, unknown kind, missing or non-positive threshold, `max_cards` below 2.
One further row proves a coefficient error from card 2's parse reaches `Active`'s caller naming `batcher.yaml`; the three coefficient cases stay in card 2.

### [NIT] Card 10 misdescribes why the worktree config resolves

**Location:** `_lyx/plan/10-calibrate-template.md`

**Issue:**
The card says the worktree's `batcher.yaml` "resolves to the template when absent".
`_lyx/config/batcher.yaml` is present in the worktree and holds only `active: ""`; the `cautious` profile resolves because the load fills the missing `profiles:` key from the template.

**Fix:**
Reword the sentence to say the worktree's `batcher.yaml` lacks `profiles:`, so the template's profiles fill in.

## Focus verdicts

- Cards 4, 7, 8 edit the same files: a defect, F1.
- Card 4 edits `registry_test.go` and card 6 deletes it: not a defect.
  Card 4 changes the `Batcher` interface, and the file's probe batcher must follow it for the package to compile until card 6 removes `register` and `lookup`; the order needs the edit.
- Intents of cards 7 and 9: not a defect.
  Card 7 states it lets nothing new through and bounds that by Plan-Gate's backward-dependency refusal, and removing the cycle condensation tightens the old behavior.
  Card 9 states what regroups and the bound (only unbegun cards, only at a triggered rebaseline).
- The identity fallbacks of cards 8 and 9: not a defect.
  Each states that a state without a partition stays on identity so the run proceeds as it started, which is the bound of Decision 10, and identity costs no more than today.
- Decisions 6 and 8 on cards 5 and 6: carried.
  Card 5 holds the `alone_above`/`budget` relationship and the one-card feasibility, card 6 holds the `max_cards` floor of 2 and the threshold positivity; no scope outside the decision record beyond F2's missing piece.
- Card 10's flags and thresholds: not a defect.
  The record licenses the median-ratio rescale, the spread bound of 3 (tunable by the Plan producer) and the no-commit outcome; the 10-card floor is a refinement of "poor fit" within that tuning latitude, since a median of a handful of cards is no fit.
- `verify:` coverage: not a defect.
  `internal/batcher` and `internal/planparser` hold no `integration`-tagged tests (every file there is untagged), and `internal/gitrepo`, `internal/websterengine`, `internal/webstercli`, `internal/loomshed` and `tools/tokencount` are in the list.
- Test economy in cards 2, 3 and 5: only the card 6 overlap, F4.
  Card 3 extends `TestHistoryReads`; card 2 and card 5 test distinct behaviors (cost function versus batchifier).
- Granularity of card 2: not a defect.
  The five symbols are entries of one card, and each is used across packages (`tools/tokencount`, `websterengine`, the batchifier), so each earns its `Create` entry.
