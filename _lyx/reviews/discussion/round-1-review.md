---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:64
    summary: Decisions 6 and 7 leave the DP undefined for a card whose own cost is under alone_above but over budget, and never say which quantity alone_above and budget compare against
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:34
    summary: Decision 1 puts the calibration report first, but the report runs the estimator in internal/batcher, which must come before it, and the record never says the estimator card precedes the report
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:130
    summary: The template weights are to come from the calibration report, but the report needs weights to run and nothing says which weights it uses or how the final ones are set from its output
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:50
    summary: Decision 4 defines a per-card message count and added context, Decision 5 prices a fork from read sets and a start-up context, so the model's inputs and the cost unit are defined in only one place each
  - id: F5
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:133
    summary: Decision 3 and the Auto-mode assumption name different base trees for a past run, and the record never says how either is found from git alone
  - id: F6
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:100
    summary: Once the partition is recorded the record does not say where the order assertion runs, nor what a rebaseline does with cards the plan gained, lost or reordered outside the unbegun tail
  - id: F7
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:74
    summary: A batcher.yaml load error can fire mid-run at every verb and at rebaseline, and no Decision bounds what it does to an already recorded partition or whether a mid-run active change takes effect
  - id: F8
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:91
    summary: A state without a recorded partition recomputes with whichever profile is active, which can regroup already begun cards when the operator switched profiles during an in-flight run
  - id: F9
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:89
    summary: Decision 10 lists status among the verbs that recompute batches, but status never forms batches, and the enumeration is a perishable list
  - id: F10
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:102
    summary: Decision 12 pins a commit hash and repeats the webster-delete-ordering split that Scope already states
---

### [BLOCKING] DP result undefined for a card over budget on its own

**Location:** `_lyx/discussion/decision-record.md` Decisions 5, 6 and 7 (lines 57, 64, 70).
**Issue:** Decision 6 makes a card whose single-card estimate exceeds `alone_above` a batch of its own, and bounds every batch by `budget`.
Decision 7 takes the minimum over contiguous splits that satisfy the hard limits.
A card whose estimate is at most `alone_above` but whose cost as its own one-card batch exceeds `budget` has no split that satisfies the limits, because every split must contain it in some batch.
The DP then has no feasible result, and the record states neither a refusal nor a rule that makes it feasible.
The record also never says which quantity `alone_above` compares against.
Decision 6 says "single-card estimate" for `alone_above` and "estimated cost" for `budget`, while Decision 4 gives a card a message count and an added context and Decision 5 defines cost as the sum of context reads.
If the two thresholds compare the same single-card cost, an `alone_above` above `budget` makes the DP infeasible for any card in between, and nothing in Decision 8's list of invalid parameters rules that configuration out.
Without a bound, the plan writer must invent the behavior of an input a real plan reaches.
**Fix:** State in Decision 6 that `alone_above` and `budget` both compare a batch's cost under Decision 5, with a single card's estimate being the cost of the one-card batch.
State that `budget` and `max_cards` limit only batches of two or more cards, so a one-card batch is always feasible and the DP always has a result.
The bound: a card over `budget` runs alone exactly as under the identity batchifier, so no card costs more than today.
Add to Decision 8 that `alone_above` greater than `budget` is not rejected, since it is harmless under that rule, or that it is rejected; pick one and state it.
Add one acceptance criterion row for a card between the two thresholds.

### [MEDIUM] Calibration first in the card order cannot hold without the estimator before it

**Location:** `_lyx/discussion/decision-record.md` Decisions 1, 3 and 4 (lines 34, 43, 50), Acceptance criteria line 150.
**Issue:** Decision 1 makes the tokencount per-fork report and the calibration report the plan's first cards.
Decision 3 and the acceptance criteria have the calibration report call the same estimator function the live batchifier calls, and Decision 4 puts that estimator in `internal/batcher`.
A card that targets the calibration report `Uses` the estimator, and Plan-Gate refuses a card that uses a target created by a later card.
So the estimator card, and the size-source interface it needs, must come before the calibration report.
Read literally, Decision 1 puts the report before anything of `internal/batcher`, which the plan cannot satisfy.
The "before the batchifier groups anything" half is satisfiable, since the estimator is not the grouping cost model, but the record does not say that is what it means.
**Fix:** Reword Decision 1 to: the per-fork report comes first, then the estimator and size source, then the calibration report, and all three precede the cost batchifier, profiles and webster changes.
Keep the rationale about tuning weights before any profile groups cards.

### [MEDIUM] Weights are circular between the calibration report and the template profile

**Location:** `_lyx/discussion/decision-record.md` Decision 5 (line 61), Auto-mode assumptions (line 130), Acceptance criteria.
**Issue:** The Auto-mode assumptions set the template's cost-profile weights from the calibration report produced by this task's own early cards.
The report runs the estimator, and the estimator takes its coefficients from a profile (Decision 5), so the report needs weights before it has produced any.
The record does not say which coefficients the report applies, whether it takes a profile as input, or how the final weights follow from its output.
Fitting by regression inside the batcher is rejected in the support log, so the setting is by hand or by a simple rule, and the record names neither.
The report also reads token transcripts that live only on the operator's machine, so a card cannot fix the numbers at plan time.
**Fix:** In Decision 3, state that the report takes the coefficients of a named profile from `batcher.yaml`, and in the auto-mode assumption state that the template ships initial coefficients and the weights card sets them from the report's per-card table by one stated rule: a single scale factor, the median measured-to-estimated ratio, applied to the cost-model units.
State that the report is run once on the operator's transcripts and the resulting numbers are committed in the template; a fit that cannot be produced leaves the initial coefficients and the profile inactive, which the existing assumption already covers.

### [MEDIUM] Decisions 4 and 5 describe two different models

**Location:** `_lyx/discussion/decision-record.md` Decisions 4 and 5 (lines 50 to 62).
**Issue:** Decision 4 says a card yields an estimated message count and an estimated context it adds.
Decision 5 prices a segment from each card's messages, a start-up context, and the union of the read sets of cards 1..k, so shared files count once.
A card's added context in Decision 4 is a single number, but under Decision 5 the context a card adds depends on the cards before it in the segment, so no per-card number states it.
The estimator must therefore return each card's read set, files with line counts, not a scalar context.
The record also defines the coefficients (messages per target, per test file, per `Uses` entry, context per line, package weight) only in Decision 5, while Decision 4 lists the signals without saying that they feed those coefficients, and how a package contributes to a read set (as a weighted pseudo-file or through its files) is open.
The cost unit is not named either, so `budget` has no stated scale relative to tokencount's weight figure that the calibration measures against.
**Fix:** Rewrite Decision 4's last sentence so a card yields its estimated message count and its read set, a map of file to weighted size, with the package and test package as named entries.
In Decision 5 state that segment cost is computed from those two outputs and the profile's coefficients, that the package entries carry the package weight directly, and that the unit is the model's own context-token unit, which the calibration report scales to tokencount's weight.

### [MEDIUM] Two phrasings pick a past run's base tree

**Location:** `_lyx/discussion/decision-record.md` Decision 3 (line 44) and Auto-mode assumptions (line 133).
**Issue:** Decision 3 takes file and test sizes from the warp tree the run's first batch started from.
The Auto-mode assumption takes the warp tree just before the run's first card commit.
The support log's Q6 repeats the second phrasing.
They coincide only if nothing lands on the warp branch between the start of the first batch and its first card commit, and the record does not claim that.
Neither says how the tree is found from git alone, since the run's `state.json` is not in git.
The plan is anchored differently again, on the first Webster fork.
**Fix:** Keep one rule in Decision 3: the plan is the last `loom: plan artifacts for <slug>` weft commit before the run's first Webster fork, and the base tree is the parent of the first commit the run's first batch made on the task branch.
Delete the Auto-mode line and make Q6 point to Decision 3 by number.

### [MEDIUM] Order assertion and plan changes under a recorded partition are unplaced

**Location:** `_lyx/discussion/decision-record.md` Decisions 10, 11 and 12 (lines 87 to 105), Constraints line 122.
**Issue:** Decision 12 says the assertion is raised "wherever batches are formed today".
Under Decision 10 batches are formed only at first init, at `validate` with no state and at a rebaseline tail re-batch; every other verb maps recorded ids onto the plan, so the sentence no longer names a place.
Decision 11 re-batches "the cards after the last begun batch", but a rebaseline can find the plan with a card added, removed or moved ahead of that point, or with a recorded id gone.
The record names one such refusal only as an example in Constraints, not in a Decision, and none of the other cases.
The existing fingerprint guard refuses a plan that changed outside a rebaseline, which covers the non-rebaseline half, but the record does not say it relies on that.
**Fix:** In Decision 12 state that the assertion runs on the partition when it is computed (first init, validate with no state, rebaseline tail) and on the mapped batches at every verb that maps a recorded partition, over the derived edges, with the recorded order as the execution order.
In Decision 11 state that every current-plan card up to and including the last card of the last begun batch must be exactly the recorded card ids of the batches up to there, in order, else the rebaseline refuses with the existing card-set-changed refusal, naming `lyx webster rebaseline` or a fresh run as the way forward.
State that a plan edited outside a rebaseline is refused by the existing fingerprint guard, so the recorded partition is never re-derived from an unaccepted plan.
Cards added, removed or reordered after that point are accepted and re-batched.

### [MEDIUM] A mid-run load error has no stated bound on the recorded partition

**Location:** `_lyx/discussion/decision-record.md` Decisions 8 and 11 (lines 74 to 99).
**Issue:** `batcher.Active` runs for every `lyx webster` verb in the CLI's pre-run wiring, and the Batchifier and Webster rows both map its error to Stuck.
Decision 8 adds load errors (unknown profile, unknown kind, invalid parameter), and Decision 11 re-batches with "the then-active profile", so a profile edited mid-run can be invalid at any verb or at the rebaseline.
The record does not say that a verb's load error leaves the recorded partition unchanged, whether an edit of `active:` between verbs changes anything other than the next rebaseline tail, or that a failed rebaseline re-batch leaves state untouched.
**Fix:** In Decision 8 or 11 state that the recorded partition is only ever replaced by a successful first init or rebaseline; a load error at any verb refuses before the verb touches state, naming `batcher.yaml` as the way forward; a rebaseline whose profile does not load refuses and leaves state unchanged; and a mid-run change of `active:` takes effect only at the next rebaseline tail re-batch, the recorded profile name staying that of the partition in force.
Add one acceptance row for the rebaseline-with-invalid-profile case.

### [MEDIUM] Legacy-state fallback is unbounded under a non-identity profile

**Location:** `_lyx/discussion/decision-record.md` Decision 10 (line 91).
**Issue:** "A state written before the partition field existed recomputes as today" is a path around the recorded partition.
Under a cost profile it would compute a new partition from the current tree for a run whose begun batches ran one card each, so the begun-batch guard would refuse or the batches would regroup, which is the failure Decision 10 exists to prevent.
Only the deploy-time default of identity makes it safe today, and nothing stops the operator switching profiles while such a run is in flight.
**Fix:** State in Decision 10 that a state without a recorded partition always batches with the identity batchifier, whatever profile is active; the fallback can then only reproduce what that run was started under.
Add that to the acceptance criterion that covers the legacy state.

### [LOW] Decision 10 lists a verb that forms no batches

**Location:** `_lyx/discussion/decision-record.md` Decision 10 (line 89).
**Issue:** The list names `status` among the verbs that recompute batches, but `internal/webstercli/status.go` never touches the batcher.
The list also names current verbs one by one, which goes stale as verbs change.
**Fix:** Replace the list with "every verb that forms batches from the plan today".

### [NIT] Decision 12 carries a hash and repeats the Scope split

**Location:** `_lyx/discussion/decision-record.md` Decision 12 (lines 102 to 105).
**Issue:** The bound cites commit `d536a1a4e`, a perishable pin, where the refusal's name is the stable reference.
The last sentence, that the plan spec and template part stays with `webster-delete-ordering`, restates what Scope's Out list says.
The Rationale's operator rule and date stay, since they carry the reason.
**Fix:** Say "Plan-Gate's backward-dependency refusal" without the hash, and drop the closing sentence, keeping the takeover half of it.

## Relocation pass

The completeness-before-leanness test found nothing else to move.
The Decision rationales carry a reason for the decision, not deliberation, and the `Notes for the plan writer` carry no rejected alternatives.
The Decision 10 and 11 rationales overlap the support log's first rejected alternative, but each states why the decision holds, which `Plan-Write` needs and never reads from the support log.
The one extraction is F10's repeated sentence, whose content already lives in Scope.
