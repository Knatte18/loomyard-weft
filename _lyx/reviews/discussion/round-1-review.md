---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D4, second bullet
    summary: D4 replaces today's spawn trigger with the bench's fork rules, which never carried the spawn mechanics the cluster audit enforces, so a fanned round loses the fork subagent type, the single-message spawn and the no-name rule
    fix: In D4's second bullet, replace "replacing today's spawn trigger, boilerplate and holistic paragraph" with "replacing today's boilerplate and holistic paragraph", and add a sub-bullet "the spawn mechanics of today's spawn trigger stay in the fanned part- every fork spawned in one message through the Agent tool with subagent_type fork, one per listed lens, never with a name - since the cluster audit fails a named spawn and a fork count unequal to the lens count". Add to the first Acceptance criterion that the fanned cluster-rules part carries those spawn mechanics.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D3, second bullet; Acceptance criteria, sixth bullet
    summary: D3 refuses an unmigrated layer holding fan_review at load, but fan_review is a local-admitted key that a task's run_settings can rewrite into the local layer after every reconcile, and configengine already retires keys by dropping them at load
    fix: Replace D3's sentence "A layer still holding the key unmigrated is refused at load with `lyx config reconcile --apply` as the way forward." with "`fan_review` is declared in loom's `ConfigSpec` `RetiredKeys` with `CategoryNone`, so every load drops it from the stacked document and a layer not yet reconciled, a local layer or a task's `run_settings` included, still loads; reconcile prunes it." Drop the load-refusal spec row this sentence implied, and in the sixth Acceptance criterion add "and a layer still holding `fan_review` loads with the key ignored".
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D2, fifth bullet; D7; D10
    summary: D2's bound that a hub config change takes effect at the next run, never mid-run, is false, because every loom run and step verb re-wires and calls LoadConfig and ResolveReview, so an edit between steps or across a pause can give one round's review half, fix half or resumed half different fans
    fix: In D2's fifth bullet, replace "so a bad later-round name fails before round 1, and a hub config change takes effect at the next run, never mid-run" with "so a bad later-round name fails before round 1". Add a D2 bullet- "A round's reviewer fan and fixer fan are fixed when its review half starts- the review half writes both resolved names into the round's usage record, and the fix half, a resumed half and the Bouncer's judgment of that round read them from there, never from loom.yaml; a hub edit reaches only rounds whose review half starts after it." Add an Acceptance criterion- "With the fan list changed between a round's review half and its fix half, the fix half runs the fixer fan the review half recorded."
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D7; Acceptance criteria, last bullet; Notes for the plan writer, fourth bullet
    summary: No Decision says how a fixer fan reaches the fixer, whether through the BurlerFix recipe row, which takes no fan and rejects unknown profile keys, or through the lyx burler run profile, whose only fan key is cluster-fan, so the standalone criterion and the conflict check rest on no Decision
    fix: Add a D7 bullet- "The burler profile gains a `fix-fan` key, read by `lyx burler run` and by the recipe's BurlerFix profile; the wiring hands the round's `discussion_fix_fan` or `plan_fix_fan` entry to the segment's BurlerFix row, as it hands the reviewer fan to the review row; a fix fan set both in loom.yaml and in the recipe's profile is refused at construction with the way forward to remove `fix-fan` from the recipe's profile for that row, and a spec row lands with it." Then cut the fourth plan-writer note, which the bullet now carries.
  - id: F5
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md Open risks, first bullet; D5 Rationale
    summary: The record says the reviewer fan rests on the bench and names only the Plan fan and two fixer lenses as unbenched, but the bench report lists the walkthrough lens, the widened existing-paths and the dropped off-lens escape as next changes that were not run, and D4 restores the board-entry line the bench dropped
    fix: Rewrite Open risks' first bullet's first sentence as "The reviewer fan rests on one bench run over one Discussion fixture, and the Discussion fan as built differs from what ran- `walkthrough`, the widened `existing-paths`, the dropped off-lens escape and the board-entry line are unbenched, as are the Plan fan and the fixer lenses `fix-proposal` and `fix-affected-set`." In D5's Rationale, replace "the bench's measured lens set" with "the bench's lens set with the report's two unrun lens changes".
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D9, first bullet; Acceptance criteria, seventh bullet
    summary: D9 names three fork violations, write, mutating git and Agent call, but the reviewer audit it reuses also fails a named spawn, and the record leaves a twill fixer's named spawn undecided
    fix: In D9's first bullet, after "calls the Agent tool", add "or is spawned with a name"; add the named spawn to the seventh Acceptance criterion's list of violations that fail the fix half.
  - id: F7
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D3, first bullet
    summary: D3 does not state the model change the retirement lets through- a fanned round, forks included, moves from fan_review's model to the segment's per-round reviewer entry, which for this hub moves round 1 from sonnet[high] to opus[medium], with no separate key left for the forks
    fix: Add to D3's first bullet- "So a fanned round, its forks included, runs on that round's entry of the reviewer list, and a hub that wants a different model for its fanned round sets that entry; with this hub's lists, retiring the hub's `fan_review` value `sonnet[high]` moves a round-1 fanned Discussion reviewer and its forks to `discussion_review`'s first entry."
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D3, third bullet; D7; Constraints, PATTERN-shuttle-provider-seam bullet
    summary: The Constraints make an engine without forks refuse a twill round at spawn, but no Decision names the key that refusal points at or its spec row, and D3's segment's reviewer key has no meaning when the segment's list is unset and the run-wide review list serves
    fix: In D3's third bullet, replace "names the segment's reviewer key" with "names the key the round's reviewer model came from (`discussion_review` or `plan_review` when set, else `review`)". Add a D7 bullet- "A twill fixer refused by an engine without fork support names the key its model came from (`discussion_fix` or `plan_fix` when set, else `fix`) and the segment's fixer fan key, with the way forward to move the fixer to an engine with forks or set that round's fixer fan entry to the empty string; its spec row lands with it."
---

### [BLOCKING] Fanned fork rules drop the spawn mechanics the audit enforces

**Location:** `_lyx/discussion/decision-record.md` D4, second bullet.

**Issue:** D4 has the fanned part carry the bench's fork rules "as the only fork rules, replacing today's spawn trigger, boilerplate and holistic paragraph".
Today's spawn trigger, in `clusterRulesBlock` (`internal/burlerengine/prompt.go`), is the only text telling the reviewer to spawn every fork in one message through the Agent tool with `subagent_type: "fork"` and never to pass a `name`.
The bench's "Fork rules for this round" (`docs/research/burler-fan-tuning-bench-2026-10-16/setup/iter1/burler-step-review.md`) carry none of that.
They were written to sit under the Go block and "replace ... wherever they differ", so the bench never ran without those mechanics.
`auditClusterRound` (`internal/burlerengine/cluster.go`) fails the half hard on `audit.NamedSpawns > 0` and on a fork count unequal to the lens count.
A reviewer that spawns named forks, or non-fork subagents the audit does not count, therefore fails every fanned round.
The prompt built as D4 states would lose the instruction that keeps the audit passing.

### [MEDIUM] Load refusal of `fan_review` is a dead end and duplicates the retired-key mechanism

**Location:** `_lyx/discussion/decision-record.md` D3, second bullet; Acceptance criteria, sixth bullet.

**Issue:** `fan_review` is `CategoryNone` in loom's `ConfigSpec` (`internal/loomengine/config.go`), so a local layer and a task's `run_settings` may hold it today.
`configengine`'s doc (`internal/configengine/doc.go`) states that Seed-Child rewrites the pair's local layer to equal the task's run settings.
A task whose `run_settings.loom` holds `fan_review` would get the key written back into the local layer after every reconcile, and the stated way forward, `lyx config reconcile --apply`, cannot remove it from the board.
The run is refused again at its next load, so the way forward makes no progress from the state the refusal leaves.
`configengine` already has the retirement mechanism this needs: a `Spec`'s `RetiredKeys` are dropped from the stacked document on every load, "so a layer not yet reconciled still loads", and reconcile prunes them.
`internal/websterengine/config.go` and `internal/shuttleengine/config.go` already retire keys this way.

### [MEDIUM] The "never mid-run" bound on a hub config change does not hold

**Location:** `_lyx/discussion/decision-record.md` D2, fifth bullet; D7; D10.

**Issue:** D2 claims a hub config change "takes effect at the next run, never mid-run".
`loomCLI.armAt` (`internal/loomcli/arm.go`) calls the full `wire` for every non-lightweight verb.
`wire` (`internal/loomcli/wiring.go`) calls `loomengine.LoadConfig` and `loomengine.ResolveReview` each time.
Each `lyx loom` step or resume re-reads `loom.yaml`, so the orch editing the hub config while a run is paused or between steps changes the fans mid-run.
Walking the focus case `discussion_fan: [review-discussion, ""]` / `discussion_fix_fan: [fix-discussion, ""]` with an edit between round 1's review half and its fix half:
the review half ran fanned, and the fix half re-resolves and can run `single` or a different fixer fan.
A resumed review half can also run a fan other than the one its usage record names, and D10's judge then asks about the wrong next-round fan.
For one unchanged config, D2, D7 and D10 do pick the same entry for the same round.
The defect is the false bound and the unstated behaviour of a round already in flight.

### [MEDIUM] No Decision carries how a fixer fan reaches the fixer

**Location:** `_lyx/discussion/decision-record.md` D7; Acceptance criteria, last bullet; Notes for the plan writer, fourth bullet.

**Issue:** `burlerFixEntry` (`internal/shedrecipe/entries_burler.go`) is documented as "The row takes no cluster fan, since the fixer runs no fan".
Its profile parser accepts only `target`, `fix-scope` and `language-skills` and rejects every other key.
`lyx burler run`'s profile (`internal/burlercli/run.go`) has one fan key, `cluster-fan`, which selects the reviewer fan.
The criterion "`lyx burler run` with a profile naming a fixer fan runs the twill fixer standalone" and the plan-writer note about the recipe-profile conflict check both presuppose a profile key and a wiring path that no Decision names.
Since notes are never a completeness requirement, the criterion is the only place this requirement exists.
The plan writer would have to invent the key name, its reach into the BurlerFix row, and the conflict refusal with its way forward and spec row.

### [MEDIUM] What the bench measured is misstated

**Location:** `_lyx/discussion/decision-record.md` Open risks, first bullet; D5 Rationale.

**Issue:** The bench report's "Next changes if the fan is pursued (not run)" lists the walkthrough lens, the widened `i1-existing-paths` and the dropped off-lens BLOCKING escape.
D5 and D4 build all three.
The bench also dropped the board-entry line, because the board entry was a fasit file there, and D4 restores it.
Open risks names only the Plan fan, `fix-proposal` and `fix-affected-set` as unbenched.
D5's Rationale calls the set "the bench's measured lens set", though two of its four lenses and the fork rules' off-lens rule never ran.
The operator reads Open risks to judge what real runs must confirm, and this one understates it.

### [LOW] D9's fork violation list omits the named spawn

**Location:** `_lyx/discussion/decision-record.md` D9, first bullet; Acceptance criteria, seventh bullet.

**Issue:** D9 audits the twill fixer "with the reviewer's per-fork violation checks" and lists write, mutating git and Agent call.
`auditClusterRound` also fails on `NamedSpawns`, an audit-wide check that is not per fork.
The record decides the count check (warn) and the three per-fork checks (fail), but leaves the named-spawn check undecided for the fixer.
Otherwise D7's "no report" rule, D9's count warning and the usage record's counted forks agree on all four cases the focus names:
a fork that writes fails the half; zero forks warn and record zero; a second fork pass warns on the count and records the larger number; lost forks are named "no report" and reach the audit as forks without a returned report.

### [LOW] D3 leaves the model change it lets through unstated

**Location:** `_lyx/discussion/decision-record.md` D3, first bullet.

**Issue:** `resolveSegmentModels` (`internal/loomengine/review.go`) replaces a fanned segment's reviewer list outright with `fan_review`.
After D3, a fanned round and its forks run on the segment's per-round reviewer entry.
In this hub's `loom.yaml` that moves a round-1 fanned Discussion reviewer from `sonnet[high]` to `discussion_review`'s first entry, `opus[medium]`, with four forks inheriting it.
Auto-mode assumptions mention dropping `fan_review: sonnet[high]` but not that the fanned round's model now follows the reviewer list, and no key is left for a fork model different from that round's reviewer.
The orch configures the hub after landing and needs that stated.

### [LOW] Engine refusal keys for the reviewer fallback and the twill fixer are unnamed

**Location:** `_lyx/discussion/decision-record.md` D3, third bullet; D7; Constraints, `PATTERN-shuttle-provider-seam` bullet.

**Issue:** `burlerReviewEntry` sets `opts.ReviewKey` to `fan_review` for a fanned row so the engine refusal names that key.
D3's replacement, "the segment's reviewer key", has no single value when `discussion_review` is unset and the run-wide `review` list serves.
The Constraints add a new refusal, an engine without forks refusing a twill round at spawn, but no Decision names the key it points at, its way forward or its spec row.
`PATTERN-refusal-way-forward` requires all three.

## Rejected

- D4 and D5, a finding outside every lens: D4's third sub-bullet gives the handler's holistic review what no lens owns, and the bench boilerplate tells each fork that the handler reviews holistically, so dropping the off-lens escape is bounded and consistent; not a defect.
- D7's one-fork-pass rule as a prompt-only bound: D9 states its enforcement as a count warning and the usage record makes a violation visible, so the bound is stated, if soft.
- Relocation: D5's and D8's rationales are short pointers to the bench and the moved lens pairing; neither carries narrative that the plan writer does not need, so no move is warranted.
- A Decision editing this task's board entry: none does; the Constraints' documentation-lifecycle bullet leaves the body unchanged.
