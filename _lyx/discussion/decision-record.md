# Decision record — fan-prompt-tuning

## Goal

Put the Burler lens fan and a twill fixer into production for Discussion-Review and Plan-Review, so one Burler round's review and fix are more thorough and fewer rounds are needed.
The reviewer's lens set and the fixer's self-check lens set are designed together per perch, from the 2026-10-16 benches (`docs/research/burler-fan-tuning-bench-2026-10-16/`, `docs/research/burler-twill-fixer-bench-2026-10-16/`) and the late-BLOCKING classification in the board entry.
Both are selectable per round, so the hub can pay for them in round 1 only.

## Scope

In:

- The reviewer's fan prompt: the explore step's pre-read for the fork reviewers and the review step's fork rules, moved from Go prose into stencils.
- Two new reviewer fans with their lenses in the embedded `burler.yaml` template: one for Discussion-Review, one for Plan-Review.
- A fixer mode per round: `single` (today's fixer) or `twill` (the fixer checks its own change set with read-only forks before its fixer-report), with two fixer fans and their lenses.
- Per-round lists for `discussion_fan`, `plan_fan` and the two new fixer keys, in the form of the per-segment model keys.
- Retiring `fan_review`, which a per-round fan makes ill-defined.
- The fixer's fork audit, and the fixer's fan, lenses and fork count in the round's usage record.
- The Bouncer judge's lens-exclusion prompt keyed to the next round's fan.
- The refusal-spec rows the changed and new refusals touch.
- Docs: `internal/burlerengine/doc.go`, `internal/loomengine` config docs, `internal/shedadapters/doc.go` where the per-round fan changes what it says.

Out:

- The hub's own `loom.yaml`: the orch sets it after landing (operator decision, 2026-10-16); this task ships the keys with defaults that leave today's behaviour unchanged.
- Webster-Review and Merge-Review: no reviewer fan and no fixer mode; their subject exists only through read-only git a fork may not run, and the operator judged the twill fixer probably not needed for code.
- Another bench: real runs are the reference (operator decision, 2026-10-16).
- A reviewer that forks its own fixer, or a fix done in a fork (stages 1-4 of the fork-review audit).
- Removing the dormant `standard`, `full`, `discussion` and `plan` fans and their lenses from the template.
- Proposal E from the board entry (lyx extracting the target's citations as the manifest's start) and the reviewer code pack of `review-inline-code`: the operator's decision builds the master's own pre-read, and the pack is that entry's.

## Decisions

### D1 — Operator decisions this record carries

All of 2026-10-16:

- The lens fan goes into production, with the lens set configured per review segment.
- Discussion-Review gets the bench's lenses with `existing-paths` widened to readers and a walkthrough lens in place of `way-forward`, plus the bench's pre-read manifest and fork rules without the off-lens BLOCKING escape (built in D4, D5).
- Plan-Review gets its own lenses, based on the Plan findings of the board entry's late-BLOCKING classification (D6).
- The fixer gets a mode, `single` or `twill`, per perch in `loom.yaml`; the default is `single` (D2, D7).
- The reviewer's and the fixer's lens sets are designed together per perch so they complement each other (D8).
- A defect in the reviewer's own proposed fix is caught at reviewer level or by a lens that challenges the proposal itself (D8).
- The lens-fan key and the fixer-mode key of each perch take one value or a per-round list, in the form of the per-segment model keys, the last entry serving every later round.
  The hub's standard setup is the fan and the twill fixer in round 1, and solo review with the single fixer from round 2; the orch sets the hub config after landing (through `ly:orch`).

### D2 — Per-round fan and fixer keys in `loom.yaml`

- `discussion_fan` and `plan_fan` keep their names and take either one fan name or a YAML list of fan names; entry N serves round N and the last entry every later round, as `discussion_review` does.
  An empty entry runs that round solo.
- Two new keys, `discussion_fix_fan` and `plan_fix_fan`, take the same shape: an empty entry is the `single` fixer for that round, a fan name is the `twill` fixer with that fan's lenses.
- Template defaults are `""` for all four keys, so a hub that sets nothing runs solo review and the single fixer in every round.
  The operator's "default is `single`" covers the fixer keys; the fan keys keep today's solo default, and the hub's standard setup is the orch's config, not a template default.
- An empty list (`[]`) is refused with the way forward to write `""`.
- Every non-empty entry of every list is resolved through `burlerengine.ResolveFan` when the run resolves its review settings at run start, so a bad later-round name fails before round 1, and a hub config change takes effect at the next run, never mid-run.
  A refusal names the key, the entry's position and the fan, and gives the way forward: define the fan in `burler.yaml` or set the entry to `""`.
- All four keys are closed to the local layer, as `discussion_fan` and `plan_fan` are today, since a lens set is what every fork is given.
- The refusal-spec rows for `discussion_fan` and `plan_fan` are widened to the fixer keys and to list entries, and new rows land for the empty list.
- Webster-Review and Merge-Review get no fixer key.

Rationale: the operator's per-round decision, in the grammar the per-segment model keys already use, so the hub's standard setup is `discussion_fan: [review-discussion, ""]` and `discussion_fix_fan: [fix-discussion, ""]`.

### D3 — `fan_review` is retired

- A round's reviewer runs on its segment's own reviewer list (`discussion_review`, `plan_review`, else `review`), fanned or not, and the forks inherit that model.
- `MigrateConfig` drops `fan_review` from every `loom.yaml` layer reconcile migrates, with one reconcile line saying so, as it retired `skip_merge_review`.
  A layer still holding the key unmigrated is refused at load with `lyx config reconcile --apply` as the way forward.
- A fanned reviewer's engine refusal names the segment's reviewer key, not `fan_review`.
- The refusal-spec rows that name `fan_review` (the model-spec list row and the cluster-fan engine-requirement row, whose way forward sets `fan_review`) change to the segment's reviewer key.

Rationale: with a per-round fan, "replace the segment's review list when the segment is fanned" has no single meaning, and the per-round reviewer list already expresses a different model for a fanned round.

### D4 — Reviewer fan prompt lives in stencils

- The explore step gets an optional marker that renders, in a fanned round only, a new stencil part with the bench's "Reading for the fork reviewers" section (`setup/iter1/burler-step-1-explore.md`), with these changes:
  - the checklist's existing-paths line also names the code that reads each state a decision changes;
  - the board-entry line of fan-lab variant D stays: the board entry of the task the target belongs to, through `lyx board get`, since no production segment's fasit carries it;
  - the manifest never ends the turn: the reviewer goes straight on to the spawn in the same turn.
  A solo round's explore step keeps its bytes.
- `clusterRulesBlock`'s Go prose moves into stencil parts read through `stencilstore`: a solo part (today's one line) and a fanned part, filled with the lens list.
  The fanned part carries the bench's fork rules (`setup/iter1/burler-step-review.md`) as the only fork rules, replacing today's spawn trigger, boilerplate and holistic paragraph, with these changes:
  - the off-lens BLOCKING escape is dropped: a fork reports only what its lens's procedure produces;
  - each fork prompt is the boilerplate, then `Your lens: <name>` and the lens text, and nothing else;
  - the handler's holistic review owns what no lens owns: the target against its fasit and board entry, the rubric's leanness and relocation rules, architecture and PATTERN-fit;
  - the consolidation paragraph is today's, unchanged, so `origin:` tags, `## Rejected` and the strict parse still hold.
- `clusterRulesBlock` survives only as the renderer of those parts.

Rationale: the bench's text is the measured one, and the Go prose is a standing departure from `PATTERN-stencil-ownership` that this move removes.

### D5 — The Discussion-Review fan, `review-discussion`

Four lenses, each a procedure that starts its final message with its own list, added to the embedded `burler.yaml` template; the texts are the bench's `i1-*` texts (`setup/iter1/burler.yaml`) except where stated:

- `interactions`: the bench text.
  Every pair of decisions that act on the same state, file, key, command, ordering or invariant, each run with the other in force; list of pairs, held or broken.
- `existing-paths`: the bench text, widened.
  For each state, setting, file or invariant a decision introduces or changes, every existing code path that writes, resets, reapplies or reads it, and for a reader, whether it still reads the state right under the new behaviour; a path that undoes or misreads a decision is a design finding about that decision, never a missing call-site list.
- `bound-breaker`: the bench text.
  Every bound, guard, guarantee, ordering promise and claim about existing code, attacked with a concrete input, interleaving or path one hop past what the record cites.
- `walkthrough`, new, replacing `way-forward`: for each decision, take one concrete case and execute its mechanism step by step, with the data each step is handed by the step before it, and check the end state against the decision's own stated purpose; list of decisions, case used, purpose met or not.

No `generic` lens: the handler's holistic review is the general pass.

Rationale: the bench's measured lens set; `existing-paths` alone found the late BLOCKING, `way-forward` was the noisiest, and the two changes are the report's "next changes".

### D6 — The Plan-Review fan, `review-plan`

Three lenses, one per class of the Plan rows in the board entry's late-BLOCKING table, each a procedure with its own list, complementing the writer's `twill-plan` lenses instead of repeating them:

- `affected-set`: for each output, key, string, file shape or behaviour a card changes, search the whole repository for everything that reads or asserts it (tests, callers, docs, fixtures), and list per change the set found and the cards that cover it; a card that covers part of the set is one finding about the method, with the members it missed.
  Evidence: `config-layers-run-settings` rounds 4 and 5.
- `guard-reach`: for each guard, refusal, branch or test row a card adds, trace a real entry point's sequence of states to it, and say whether the state it guards can occur there and whether the guarded site recurs; list per guard, reachable or not.
  Evidence: `logging-cleanup` card 9.
- `way-forward`: for each refusal, stop, reset or recovery step a card adds, the next step it names and whether that step can succeed from the state the refusal leaves; list per refusal, progresses or not.
  Evidence: `webster-dead-ends` round 4.

Rationale: the operator asked for Plan lenses based on the Plan late findings, which fall into these three classes; a lens with no Plan evidence behind it is left out.

### D7 — The twill fixer

- A round whose fixer fan entry is non-empty spawns its fixer with fork subagents authorized, and the fix step renders a new burler stencil part, the self-check, before the fixer-report rule; a `single` round's fix step keeps its bytes.
- The self-check part is the bench's block (`setup/selfcheck-block.md`) with these changes:
  - the rubric path is a new deployed stencil, the bench's `setup/fix-check-rubric.md`;
  - the lens list is the round's fixer fan;
  - step 4 judges each fork finding by the fix step's own three dispositions (Dispute, Fix with your own change, Fix as proposed) by pointing at the fix step's Fix-everything rule, not by restating them;
  - the fixer records the self-check in a `## Self-check` section of the fixer-report, one line per fork finding, naming a fork that returned no report.
- One fork pass per fixer session stays: a fixer that has made its spawn call spawns no further forks, whether it is re-prompted by the fix half's gate or resumed after its process died; a fixer resumed after its forks were lost judges the findings it holds and names each lost fork as "no report".
- The self-check runs in every fixer session of a twill round, including a gate-findings-only fix; a round whose fixer is skipped on an empty review runs no fixer and no self-check.

Rationale: the bench's block ran as designed in every twill run with fewer fix-introduced findings and wrong dispositions than today's fixer, at a cost the operator accepted for round 1.

### D8 — The fixer fans, `fix-discussion` and `fix-plan`

Each fan has four lenses over the change set alone, added to the embedded template:

- `fix-consistency` (both): the bench's `consistency` lens: every sibling decision, card, criterion, row or cross-reference the changed text touches or depends on still agrees with it.
- `fix-claims` (both): the bench's `claims` lens: every sentence the fix added or rewrote that asserts a bound, guarantee, behaviour, count or fact about the code holds at HEAD, claims taken over from the reviewer's proposal included.
- `fix-proposal` (both), new: for each finding the fixer fixed as proposed, challenge the proposal itself: build the concrete case where the proposed change is wrong, or name what it breaks elsewhere in the artifact, rather than checking only that it was carried out.
  This is the check of the reviewer's proposal D1 asks for; no reviewer lens makes it.
- `fix-way-forward` (Discussion): the bench's `way-forward` lens: every refusal, error, next step, row or test the fix introduced can be carried out from the state it names and makes progress.
- `fix-affected-set` (Plan), new: for each finding whose premise is a set (tests, callers, sites under one key), rerun the finding's search over the whole repository and check the fix covers every member, not only those the review named.

Rationale: the bench's three lenses ran clean, but the fix-introduced defects left in its twill arm came from the reviewer's proposal, which the forks took as given; Plan's late findings show fixes covering part of a set.

### D9 — Fork audit and usage record for the fixer

- A twill fixer's run is audited with the reviewer's per-fork violation checks: a fork that writes, runs a mutating git command or calls the Agent tool fails the fix half and stops the run as a failed fanned review half does, with a refusal naming the fork and its violation and the way forward: inspect the target's working-tree changes, revert what the fork wrote, and resume the run.
  A spec row lands with it.
- A fork count unequal to the fixer fan's lens count is a logged warning, not an error: the fix is on disk and the next review sees it, and failing the half would redo a finished fix.
- The round's usage record gains the fixer half's fan, lenses and the number of forks the audit counted, so a twill fixer that skipped its self-check shows as zero forks in the record real runs are measured from.

Rationale: read-only forks are the self-check's premise, and a fork's write may have corrupted the fix, so it stops for the operator; a missing self-check leaves a correct fix and is made visible rather than fatal.

### D10 — The Bouncer's lens exclusions follow the per-round fan

- The judge prompt asks for `exclude_lenses`, and the facts file carries its `## Lenses` section, only when the round being judged ran a fan and the next round's resolved fan is the same non-empty fan.
- The told `ClusterExcludes` becomes a per-round answer from the segment's fan list instead of one value per row.
- The existing eligibility rules in `excludedLenses` stay: an entry counts only when the previous round's usage record names the same fan.

Rationale: `ClusterExcludes` is told once per row today, which a per-round fan list makes wrong: under the hub's standard setup the judge would be asked to trim lenses of a round that runs solo.

## Constraints

- `PATTERN-stencil-ownership`: every new prompt part (fork reading, fan cluster rules, solo cluster rule, fixer self-check, fix-check rubric) is a stencil under `contracts/stencils/burler/`, read at call time through `stencilstore`.
- `PATTERN-config-layers` and `PATTERN-config-strictness`: the new keys and the per-round lists are read through `configengine` like every loom key; `burler.yaml` stays seed-only and operator-owned, so the new lenses and fans reach a hub through the template's per-name fallback, never by rewriting its file.
- `PATTERN-binary-change-reconcile`: the `fan_review` retirement runs in `MigrateConfig` under `hubreconcile.Ensure`.
- `PATTERN-refusal-way-forward`: each new or changed refusal names its way forward, lands or changes its spec row and has a test.
- `PATTERN-model-choice-mapping`: retiring `fan_review` changes which list feeds the reviewer, not how a model-spec maps onto a spawn.
- `PATTERN-shuttle-provider-seam`: fork authorization stays `Spec.ForkSubagents`; an engine without fork support refuses a fanned or twill round at spawn, as it refuses a fanned review today.
- `PATTERN-test-economy` and `PATTERN-test-speed`: new behaviour extends the existing prompt, config, profile and usage tests as rows wherever one fits, and no test pins a stencil's or lens's static prose.
- `PATTERN-documentation-lifecycle`: the built design goes into `internal/burlerengine/doc.go` and the loom config docs; this record's board entry body is not changed.

## Auto-mode assumptions

Self-made picks the plan writer should treat as provisional, each stated in its decision:

- D2: the fixer mode is a fan-name key, empty for `single`; the fan keys keep their solo default.
- D3: `fan_review` is retired, which the operator never decided; reconcile drops the hub's `fan_review: sonnet[high]`.
- D5, D6, D8: the new fans get new names (`review-discussion`, `review-plan`, `fix-discussion`, `fix-plan`), leaving the dormant `discussion` and `plan` fans with their meaning for any profile that names them.
- D6: the Plan-Review lens set.
- D8: the proposal check sits in the fixer, not the reviewer.
- D9: fork violations stop the run; a fork-count mismatch warns.
- Scope: Webster-Review and Merge-Review get no fixer key.

## Open risks

- The reviewer fan rests on one bench run over one Discussion fixture; the Plan fan and the fixer lenses `fix-proposal` and `fix-affected-set` are unbenched.
  The after-measure over real runs is `plan-writer-finding-classes`' audit, which the operator placed after this task.
- Overlap between forks stayed high in the bench; the procedure lenses reduce it but do not remove it.
- A hub whose `burler.yaml` defines a lens named like a new one (`interactions`, `walkthrough`, ...) shadows the new text by the per-name fallback.
- `codex-engine` lists the Burler fixer among the single-session roles that spawn no forks; after this task a twill-round fixer depends on forks, so only a `single` fixer stays a Codex candidate, and a hub moving a Discussion or Plan fixer to an engine without forks sets that perch's fixer fan to `""`.
- `loom-model-key-names` plans to rename the loom model keys and lists `fan_review`; it must start from this task's key set (`fan_review` gone, `discussion_fix_fan` and `plan_fix_fan` added).

## Acceptance criteria

- A fanned round's explore step carries the fork-reading part and its review step the fanned cluster-rules part, filled with every lens of the round's fan in fan order; a solo round's two steps render the same bytes as before this task.
- A twill round's fix step carries the self-check part filled with the fixer fan's lenses and the fix-check rubric path; a single round's fix step renders the same bytes as before this task.
- The twill fixer is spawned with fork subagents authorized and a single fixer without.
- `discussion_fan`, `plan_fan`, `discussion_fix_fan` and `plan_fix_fan` each accept a scalar and a list; round N takes entry N, every round past the list's end takes the last entry, and an empty entry is solo or single for that round.
- An empty list, and a list entry naming an unknown fan, are refused at run start, before round 1, naming the key and the way forward; the four keys are refused in the local layer.
- An existing `loom.yaml` holding `fan_review` loses it on reconcile, with a reconcile line, and a second reconcile changes nothing; a fanned round's reviewer runs on the segment's own reviewer list, and its engine refusal names that key.
- A twill fixer whose fork writes a file, runs a mutating git command or calls the Agent tool fails the fix half with the refusal D9 names; one that spawns fewer or more forks than lenses, none included, logs a warning and the round goes on.
- The round usage record carries the fixer half's fan, lenses and counted forks for a twill round, and none of them for a single round.
- With a fan in round 1 and solo from round 2, the round-1 judge prompt asks for no `exclude_lenses`; with the same fan in rounds 1 and 2 it does.
- The embedded template resolves `review-discussion`, `review-plan`, `fix-discussion` and `fix-plan` to the lens names D5, D6 and D8 list, in that order.
- `lyx burler run` with a profile naming a fixer fan runs the twill fixer standalone.

## Notes for the plan writer

- The bench texts to start from are in `docs/research/burler-fan-tuning-bench-2026-10-16/setup/iter1/` and `docs/research/burler-twill-fixer-bench-2026-10-16/setup/`; the bench's `PAIRDIR/.lyx/btf/` rubric path becomes the deployed stencil's path.
- The per-round string list can reuse the scalar-or-list decoding the per-segment model keys use (`ModelSpecList`), with fan-name validation in place of model-spec parsing.
- The per-round fan reaches the review adapter as the per-round model lists do (`RowClusterFans` today holds one fan per row), and the round picks its entry where it picks its models.
- The recipe-profile conflict check in `entries_burler.go` (a fan in both `loom.yaml` and the recipe profile) applies to the fixer fan too.
- The fixer table also runs the round's gates; the fork audit and the one-fork-pass rule must hold across the gate's re-prompts of the same session.
