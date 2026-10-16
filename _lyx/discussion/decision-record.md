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
- Every non-empty entry of every list is resolved through `burlerengine.ResolveFan` when the run resolves its review settings at run start, so a bad later-round name fails before round 1.
  A refusal names the key, the entry's position and the fan, and gives the way forward: define the fan in `burler.yaml` or set the entry to `""`.
- A round's reviewer fan and fixer fan are fixed when its reviewer is spawned.
  The Discussion and Plan review halves, on the path that spawns a new reviewer only, after the live-reviewer probe finds none, write both resolved names, an empty name for a solo or single round included, into the round's usage record before the spawn, and a failed write fails the review half before any spawn.
  A resumed live reviewer keeps the names it was spawned under, and the accepted-review hand-on path writes nothing.
  The acceptance-time merge stops writing the fan from the producer's profile and writes only the lenses and the review half, so a hub edit between spawn and acceptance cannot overwrite the recorded name.
  The fix half, a resumed half and the Bouncer's judgment of that round read the names from the record, never from `loom.yaml`; a hub edit reaches only rounds whose reviewer is spawned after it.
- The record holds names exactly when it carries `fans_recorded: true`, written with `fan` and a new `fix_fan` in the pre-spawn write.
  The flag, not the `fan` key, decides, because `fan` is a plain string without `omitempty`: a missing key and an old binary's solo record both decode as `""`.
  No later writer's merge sets or clears the three keys, so the names survive every merge; a record that `mergeRoundUsage` replaces because it did not decode starts without the flag, so its names are absent, never empty.
- A Discussion or Plan fix half whose round's usage record lacks the flag, or does not decode, archives the round's review, fixer report and acceptance record together, as the fix producer's changed-review exit already does, and refuses with the way forward to `goto` the segment's review row and resume.
  The archive is what makes that way forward work: the review row then finds no accepted review, spawns a fresh reviewer and writes the names before the spawn.
  A spec row lands with it.
- The Webster, Merge and Darn review halves record no names, and their fix halves read none: they run the single fixer, and the refusal never applies to them.
- The Bouncer's one read of the names is the judged round's fan for D10's `exclude_lenses` decision.
  It degrades, never refuses: a record that is missing, lacks the flag or does not decode counts as a round that ran no fan, so the judge asks for no `exclude_lenses`, and the facts file notes the unreadable record as `computeLensFacts` notes a usage error today; `lensesRunUnderFan` keeps treating such a round as ineligible.
- All four keys are closed to the local layer, as `discussion_fan` and `plan_fan` are today, since a lens set is what every fork is given.
- The refusal-spec rows for `discussion_fan` and `plan_fan` are widened to the fixer keys and to list entries, and new rows land for the empty list.
- Webster-Review and Merge-Review get no fixer key.

Rationale: the operator's per-round decision, in the grammar the per-segment model keys already use, so the hub's standard setup is `discussion_fan: [review-discussion, ""]` and `discussion_fix_fan: [fix-discussion, ""]`.

### D3 — `fan_review` is retired

- A round's reviewer runs on its segment's own reviewer list (`discussion_review`, `plan_review`, else `review`), fanned or not, and the forks inherit that model.
  So a fanned round, its forks included, runs on that round's entry of the reviewer list, and a hub that wants a different model for its fanned round sets that entry; with this hub's lists, retiring the hub's `fan_review` value `sonnet[high]` moves a round-1 fanned Discussion reviewer and its forks to `discussion_review`'s first entry.
- `MigrateConfig` drops `fan_review` from every `loom.yaml` layer reconcile migrates, with one reconcile line saying so, as it retired `skip_merge_review`.
  `fan_review` is declared in loom's `ConfigSpec` `RetiredKeys` with `CategoryNone`, so every load drops it from the stacked document and a layer not yet reconciled, a local layer or a task's `run_settings` included, still loads.
- A fanned reviewer's engine refusal names the key the round's reviewer model came from (`discussion_review` or `plan_review` when set, else `review`), not `fan_review`.
- The refusal-spec rows that name `fan_review` (the model-spec list row and the cluster-fan engine-requirement row, whose way forward sets `fan_review`) change to that key.

Rationale: with a per-round fan, "replace the segment's review list when the segment is fanned" has no single meaning, and the per-round reviewer list already expresses a different model for a fanned round.

### D4 — Reviewer fan prompt lives in stencils

- The explore step gets an optional marker that renders, in a fanned round only, a new stencil part with the bench's "Reading for the fork reviewers" section (`setup/iter1/burler-step-1-explore.md`), with these changes:
  - the checklist's existing-paths line also names the code that reads each state a decision changes;
  - a new checklist line: for each output, key, string, file shape or behaviour the target changes, every test, caller, doc and fixture that reads or asserts it;
  - the board-entry line of fan-lab variant D stays: the board entry of the task the target belongs to, through `lyx board get`, since no production segment's fasit carries it.
  A solo round's explore step keeps its bytes.
- `clusterRulesBlock`'s Go prose moves into stencil parts read through `stencilstore`: a solo part (today's one line) and a fanned part, filled with the lens list.
  The fanned part carries the bench's fork rules (`setup/iter1/burler-step-review.md`) as the only fork rules, replacing today's boilerplate and holistic paragraph, its cap of 6 tool calls per fork included, with these changes:
  - the spawn mechanics of today's spawn trigger stay in the fanned part: every fork spawned in one message through the Agent tool with `subagent_type: "fork"`, one per listed lens, never with a `name`, since the cluster audit fails a named spawn and a fork count unequal to the lens count;
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

Rationale: the bench's lens set with the report's two unrun lens changes; `existing-paths` alone found the late BLOCKING, `way-forward` was the noisiest, and the two changes are the report's "next changes".

### D6 — The Plan-Review fan, `review-plan`

Three lenses, one per class of the Plan rows in the board entry's late-BLOCKING table, each a procedure with its own list, complementing the writer's `twill-plan` lenses instead of repeating them:

- `affected-set`: for each output, key, string, file shape or behaviour a card changes, find everything that reads or asserts it (tests, callers, docs, fixtures) from the pre-read and a search for what it lacks, and list per change the set found and the cards that cover it; a card that covers part of the set is one finding about the method, with the members it missed.
  Evidence: `config-layers-run-settings` rounds 4 and 5.
- `guard-reach`: for each guard, refusal, branch or test row a card adds, trace a real entry point's sequence of states to it, and say whether the state it guards can occur there and whether the guarded site recurs; list per guard, reachable or not.
  Evidence: `logging-cleanup` card 9.
- `way-forward`: for each refusal, stop, reset or recovery step a card adds, the next step it names and whether that step can succeed from the state the refusal leaves; list per refusal, progresses or not.
  Evidence: `webster-dead-ends` round 4.

Rationale: the operator asked for Plan lenses based on the Plan late findings, which fall into these three classes; a lens with no Plan evidence behind it is left out.

### D7 — The twill fixer

- A round whose fixer fan entry is non-empty spawns its fixer with fork subagents authorized, and the fix step renders a new burler stencil part, the self-check, before the fixer-report rule; a `single` round's fix step keeps its bytes.
- The self-check part is the bench's block (`setup/selfcheck-block.md`), whose fork boilerplate caps each fork at 6 further tool calls, with these changes:
  - the rubric path is a new deployed stencil, the bench's `setup/fix-check-rubric.md`;
  - the lens list is the round's fixer fan;
  - step 4 judges each fork finding by the fix step's own three dispositions (Dispute, Fix with your own change, Fix as proposed) by pointing at the fix step's Fix-everything rule, not by restating them;
  - the fixer records the self-check in a `## Self-check` section of the fixer-report, one line per fork finding, naming a fork that returned no report.
- One fork pass per fixer session stays: a fixer that has made its spawn call spawns no further forks, whether it is re-prompted by the fix half's gate or resumed after its process died; a fixer resumed after its forks were lost judges the findings it holds and names each lost fork as "no report".
- The self-check runs in every fixer session of a twill round, including a gate-findings-only fix; a round whose fixer is skipped on an empty review runs no fixer and no self-check.
- The burler profile gains a `fix-fan` key.
  In a shed run the wiring hands the round's `discussion_fix_fan` or `plan_fix_fan` list to the segment's BurlerReview row, beside its reviewer fan; the review half records the round's picked entry per D2, and the BurlerFix row reads the recorded name.
  A fix fan set both in `loom.yaml` and in the recipe's BurlerReview profile is refused at construction with the way forward to remove `fix-fan` from the recipe's profile for that row, and a spec row lands with it.
  `lyx burler run`, which has no shed usage record, reads `fix-fan` from its own profile.
- The fixer's spawn carries the key its model came from, as the reviewer's carries `RunOpts.ReviewKey`: `RunOpts` gains a `FixKey`, and the wiring tells each Discussion and Plan BurlerFix row a per-row key pair, its fixer model key (`discussion_fix` or `plan_fix` when set, else `fix`) and its fixer fan key (`discussion_fix_fan` or `plan_fix_fan`), beside the per-row model lists.
- A twill fixer refused by an engine without fork support names the told fixer model key and fixer fan key, with the way forward to move that fixer key to an engine with forks and resume; it adds that setting the fixer fan entry to `""` reaches only rounds whose reviewer is spawned after the edit, per D2.
  In `lyx burler run`, which resolves no key, the model key is the `shuttle.yaml` default engine, as for its reviewer today, and the fan key is the profile's `fix-fan`; the way forward is to set `engine:` in `shuttle.yaml` to an engine with forks, or remove `fix-fan` from the profile, and re-run.
  The Burler spec row quotes one wording with the two key names as its slots, so it covers both callers.

Rationale: the bench's block ran as designed in every twill run with fewer fix-introduced findings and wrong dispositions than today's fixer, at a cost the operator accepted for round 1.

### D8 — The fixer fans, `fix-discussion` and `fix-plan`

Each fan has four lenses over the change set alone, added to the embedded template:

- `fix-consistency` (both): the bench's `consistency` lens: every sibling decision, card, criterion, row or cross-reference the changed text touches or depends on still agrees with it.
- `fix-claims` (both): the bench's `claims` lens: every sentence the fix added or rewrote that asserts a bound, guarantee, behaviour, count or fact about the code holds at HEAD, claims taken over from the reviewer's proposal included.
- `fix-proposal` (both), new: for each finding the fixer fixed as proposed, challenge the proposal itself: build the concrete case where the proposed change is wrong, or name what it breaks elsewhere in the artifact, rather than checking only that it was carried out.
  This is the check of the reviewer's proposal D1 asks for; no reviewer lens makes it.
- `fix-way-forward` (Discussion): the bench's `way-forward` lens: every refusal, error, next step, row or test the fix introduced can be carried out from the state it names and makes progress.
- `fix-affected-set` (Plan), new: for each finding whose premise is a set (tests, callers, sites under one key), rerun the finding's own search, one tool call per finding, and check the fix covers every member, not only those the review named.
  When the round's set-premised findings exceed the fork's call cap (D7), the fork reruns the searches for the most severe findings first and lists each finding it left unchecked, so the fixer-report's `## Self-check` names them.

Rationale: the bench's three lenses ran clean, but the fix-introduced defects left in its twill arm came from the reviewer's proposal, which the forks took as given; Plan's late findings show fixes covering part of a set.

### D9 — Fork audit and usage record for the fixer

- A twill fixer's run is audited with the reviewer's per-fork violation checks: a fork that writes, runs a mutating git command, calls the Agent tool or is spawned with a name fails the fix half and stops the run as a failed fanned review half does, with a refusal naming the fork and its violation and the way forward: inspect the target's working-tree changes, revert what the fork wrote, and resume the run.
  A spec row lands with it.
- A fork count unequal to the fixer fan's lens count is a logged warning, not an error: the fix is on disk and the next review sees it, and failing the half would redo a finished fix.
- The round's usage record gains, beside the `fix_fan` D2 records, the fixer half's lenses and the number of forks the audit counted, so a twill fixer that skipped its self-check shows as zero forks in the record real runs are measured from.

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

- The reviewer fan rests on one bench run over one Discussion fixture, and the Discussion fan as built differs from what ran: `walkthrough`, the widened `existing-paths`, the dropped off-lens escape and the board-entry line are unbenched, as are the Plan fan and the fixer lenses `fix-proposal` and `fix-affected-set`.
  The bench ran its fork rules under today's Go fork boilerplate and holistic paragraph, which D4 removes, so the fork prompt without them is unbenched.
  The reviewer forks' call cap (D4) is unbenched against the Plan lenses, `affected-set` above all, whose sets span tests, docs and fixtures, and the fixer forks' cap (D7) against `fix-affected-set`, which spends one call per set-premised finding.
  The after-measure over real runs is `plan-writer-finding-classes`' audit, which the operator placed after this task.
- Overlap between forks stayed high in the bench; the procedure lenses reduce it but do not remove it.
- A hub whose `burler.yaml` defines a lens named like a new one (`interactions`, `walkthrough`, ...) shadows the new text by the per-name fallback.
- `codex-engine` lists the Burler fixer among the single-session roles that spawn no forks; after this task a twill-round fixer depends on forks, so only a `single` fixer stays a Codex candidate, and a hub moving a Discussion or Plan fixer to an engine without forks sets that perch's fixer fan to `""`.
- `loom-model-key-names` plans to rename the loom model keys and lists `fan_review`; it must start from this task's key set (`fan_review` gone, `discussion_fix_fan` and `plan_fix_fan` added).

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 3: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/discussion/round-3-review.md; fixer report: _lyx/reviews/discussion/round-3-fixer-report.md.
  - d2-absent-names-refusal-loops (class design, severity BLOCKING, open in rounds 3)
  - d2-bouncer-refusal-vs-tolerant-reading (class consistency, severity MEDIUM, open in rounds 3)
  - d2-prespawn-write-vs-live-resume (class design, severity MEDIUM, open in rounds 3)
  - fixer-fork-cap-vs-fix-affected-set (class design, severity MEDIUM, open in rounds 3)
  - usage-record-authoritative-without-hard-write (class design, severity MEDIUM, open in rounds 2, 3)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- A fanned round's explore step carries the fork-reading part and its review step the fanned cluster-rules part, filled with every lens of the round's fan in fan order and carrying the spawn mechanics D4 keeps; a solo round's two steps render the same bytes as before this task.
- A twill round's fix step carries the self-check part filled with the fixer fan's lenses and the fix-check rubric path; a single round's fix step renders the same bytes as before this task.
- The twill fixer is spawned with fork subagents authorized and a single fixer without.
- `discussion_fan`, `plan_fan`, `discussion_fix_fan` and `plan_fix_fan` each accept a scalar and a list; round N takes entry N, every round past the list's end takes the last entry, and an empty entry is solo or single for that round.
- An empty list, and a list entry naming an unknown fan, are refused at run start, before round 1, naming the key and the way forward; the four keys are refused in the local layer.
- An existing `loom.yaml` holding `fan_review` loses it on reconcile, with a reconcile line, and a second reconcile changes nothing; a layer still holding `fan_review` loads with the key ignored; a fanned round's reviewer runs on the segment's own reviewer list, and its engine refusal names that key.
- A twill fixer whose fork writes a file, runs a mutating git command, calls the Agent tool or is spawned with a name fails the fix half with the refusal D9 names; one that spawns fewer or more forks than lenses, none included, logs a warning and the round goes on.
- The round usage record carries the fixer fan, lenses and counted forks for a twill round, and for a single round an empty `fix_fan` with no lenses or fork count.
- With a fan in round 1 and solo from round 2, the round-1 judge prompt asks for no `exclude_lenses`; with the same fan in rounds 1 and 2 it does.
- The embedded template resolves `review-discussion`, `review-plan`, `fix-discussion` and `fix-plan` to the lens names D5, D6 and D8 list, in that order.
- `lyx burler run` with a profile naming a fixer fan runs the twill fixer standalone.
- With the fan list changed between a round's review half and its fix half, the fix half runs the fixer fan the review half recorded; with it changed before a live reviewer is resumed, the record keeps the names the reviewer was spawned under; with it changed between spawn and acceptance, the acceptance merge leaves the recorded fan unchanged.
- A review half whose fan-name write fails spawns no reviewer and fails.
- A Discussion or Plan fix half whose round's usage record lacks `fans_recorded`, an old binary's record with `fan: ""` included, archives the round's review, fixer report and acceptance record and refuses with the way forward to `goto` the review row and resume; re-stepping a round whose accepted review has such a record spawns a fresh reviewer and ends with the names recorded.
- A Discussion or Plan fix half whose round's usage record does not decode takes the same archive and refusal.
- A Webster, Merge or Darn round records no names and its fix half runs the single fixer without refusing.
- A Bouncer judging a round whose usage record is missing, lacks `fans_recorded` or does not decode asks for no `exclude_lenses`, notes the record on the facts file and does not fail.
- A twill fixer refused by an engine without fork support names its told fixer model key and fixer fan key, with the way forward to move that model key to an engine with forks; under `lyx burler run` it names the `shuttle.yaml` default engine and the profile's `fix-fan`.

## Notes for the plan writer

- The bench texts to start from are in `docs/research/burler-fan-tuning-bench-2026-10-16/setup/iter1/` and `docs/research/burler-twill-fixer-bench-2026-10-16/setup/`; the bench's `PAIRDIR/.lyx/btf/` rubric path becomes the deployed stencil's path.
- The per-round string list can reuse the scalar-or-list decoding the per-segment model keys use (`ModelSpecList`), with fan-name validation in place of model-spec parsing.
- The per-round fan reaches the review adapter as the per-round model lists do (`RowClusterFans` today holds one fan per row), and the round picks its entry where it picks its models.
- The fixer table also runs the round's gates; the fork audit and the one-fork-pass rule must hold across the gate's re-prompts of the same session.
