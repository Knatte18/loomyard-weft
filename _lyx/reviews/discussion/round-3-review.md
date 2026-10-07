---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:67
    summary: Decision 5 calls the cost unit tokencount's weight unit while the initial thresholds are in context tokens, and no rule says how the calibration card sets alone_above and budget.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:47
    summary: The per-card estimate in the calibration table, the ratio and the recorded batch estimate are never defined as the Decision 5 one-card cost, and the same estimator function is named for two different outputs.
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:34
    summary: Decision 1 claims the first three cards mean nothing groups before the weights are tuned, but the tuning card runs after the template profile and cost batchifier, and the card that owns the initial coefficients is not placed.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:181
    summary: The acceptance criterion says a rebaseline on a state without a partition re-batches nothing while Decision 11 keeps the identity batchifier for the tail.
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:105
    summary: One profile name is recorded per run but a rebaseline under a changed active profile yields a mixed partition, and the estimates and name of the kept batches are not stated.
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:68
    summary: Decision 5 rationale calls the model the operator's accepted model, but the per-fork term that makes merging pay is a self-pick the support log asks the operator to confirm.
  - id: F7
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:188
    summary: The Notes for the plan writer list the webster verbs that recompute batches, a call-site list the writer stencil says not to gather.
---

### [MEDIUM] Threshold unit and scaling rule are undefined

**Location:** `_lyx/discussion/decision-record.md:67` (Decision 5) and `_lyx/discussion/decision-record.md:149` (Auto-mode assumptions).
**Issue:** Decision 5 says the cost unit is tokencount's weight unit and, in the same sentence, that the initial coefficients and thresholds are chosen in context tokens.
`tools/tokencount/main.go` defines weight as input + 1.25 x cache writes + 0.1 x cache reads + 5 x output, so a context token read by a message is not a weight unit, and the two sentences disagree until calibration.
The calibration rule scales the start-up context, context-per-line and package-weight coefficients by the median ratio, then says `alone_above` and `budget` "are then set in the resulting weight unit" without saying by what rule.
A plan writer cannot write the tuning card's behaviour or its acceptance from that.
The missing step follows from the model: those three are the only coefficients that carry a context size, and the cost is linear in them, so every estimated cost scales by exactly the factor, and a threshold chosen in the initial unit keeps its meaning only if it scales by the same factor.
**Fix:** State in Decision 5 that cost is a sum of per-message context reads, equal to context tokens under the initial coefficients, and that calibration rescales it to weight.
State in Auto-mode that the factor multiplies exactly those three coefficients, so every cost scales by it, and that the same card multiplies `alone_above` and `budget` by the same factor and commits all the numbers.

### [MEDIUM] The estimate is never defined as a cost, and "the estimator function" names two things

**Location:** `_lyx/discussion/decision-record.md:47` (Decision 3), `:52-59` (Decision 4), `:171` (Acceptance).
**Issue:** Decision 4 defines the estimator's outputs as a message count and a read set; the cost that a fork spends is Decision 5's segment cost, which adds the start-up context and the per-fork messages.
Decision 3 and the acceptance criterion say the calibration report and the live batchifier call "the same estimator function" and the table has an "estimate" column, but a ratio of measured weight to a message count and a read set is not defined.
For the scale factor to be valid, the estimate must be the cost of the card's one-card batch, because a past fork is a one-card fork and pays the per-fork messages.
Decision 10's per-batch estimate has the same gap.
Coefficients are in fact applied once each (Decision 4 applies the message and size coefficients, Decision 5 the start-up context and per-fork messages), but nothing says so.
**Fix:** In Decision 5, name the segment-cost function as the one function both callers use, define a card's estimate as its one-card batch cost, and say which Decision applies which coefficient.
Align Decision 1, Decision 3, Decision 10 and the acceptance criterion to that wording.

### [MEDIUM] Card order contradicts the tuning rule, and the initial coefficients have no owner

**Location:** `_lyx/discussion/decision-record.md:34-37` (Decision 1), `:84-87` (Decision 8), `:149` (Auto-mode assumptions).
**Issue:** Decision 1 says the tokencount report, the estimator and the calibration report all precede the cost batchifier, "so nothing groups cards before the weights are tuned".
The weights are tuned by a late card that runs the report with the template's initial coefficients, and Decision 8 ships those with the profiles, after the calibration report.
So the tuning card follows the cost batchifier, and the stated guarantee is false as ordered; what actually keeps cards from grouping before tuning is that the cost profile ships inactive with identity the default.
The record also never says which card first provides the pieces the focus asked about.
Reading Decisions 1, 3 and 8 together: the estimator card provides the coefficients type and its parse, the calibration report can be built from that card with a fixture `batcher.yaml`, and the initial coefficients arrive only with the template in Decision 8.
Which card validates a negative coefficient is also unplaced, though the calibration report uses that parse.
**Fix:** In Decision 1, place the cards in full order: report, estimator with cost function and the coefficient parse (rejecting a missing or negative coefficient), calibration report on a fixture profile, then the profiles card with the template's initial coefficients, and the tuning card after both.
Restate the rationale as the inactive cost profile and the identity default, not card order.
Note in Decision 8 which load errors the estimator's parse already raises.

### [LOW] "Re-batches nothing" contradicts "keeps the identity batchifier for the tail"

**Location:** `_lyx/discussion/decision-record.md:181` (Acceptance) against `:113` (Decision 11).
**Issue:** Decision 11 settles the focus question: a state without a recorded partition keeps identity for the tail and records none.
The acceptance criterion words the same behaviour as "re-batches nothing", which a test writer can read as skipping the tail entirely.
**Fix:** Reword the criterion to "the tail stays on the identity batchifier and no partition is recorded".

### [LOW] Per-run profile name against a mixed partition

**Location:** `_lyx/discussion/decision-record.md:101` (Decision 10), `:91` (Decision 8), `:109` (Decision 11).
**Issue:** A rebaseline batches the tail with the then-active profile, so after a profile change the partition mixes two profiles.
Decision 10 records one profile name, Decision 8 says it "stays that of the partition in force", and Decision 11 replaces the partition.
The goal of recording name and estimate, to set a real batch's tokens against its estimate, fails if a kept batch is labelled with the wrong profile or its estimate is recomputed on the changed tree.
**Fix:** Record the profile name per batch beside its estimate; a begun batch keeps both at a rebaseline and the tail gets new ones.
Align Scope, Decisions 8, 10 and 11, and the acceptance criteria.

### [LOW] Rationale attributes the model to the operator

**Location:** `_lyx/discussion/decision-record.md:68` (Decision 5).
**Issue:** The rationale reads "the operator's accepted model".
The support log's Q16 records that without the per-fork term every merged card costs at least what it costs alone, that the term was a self-pick, and that the operator should confirm it.
The plan writer reads only the decision record and would take the saving term as settled.
The focus question itself resolves positively: merged minus apart is the per-fork messages times the start-up context, less the later cards' messages times the weight of the read set they add, so a merge can cost less exactly through the once-per-fork term.
**Fix:** Say the rationale is the operator's context-read model plus one per-fork term, the only term by which merging saves, and point at the open risk.

### [NIT] Call-site list in the notes

**Location:** `_lyx/discussion/decision-record.md:188` (Notes for the plan writer).
**Issue:** The note names the webster verbs that recompute batches.
The writer stencil says discussion does not gather call-site lists, Decision 10 already requires "every verb that forms batches from the plan", and the list is a perishable specific.
**Fix:** Drop the verb list and keep the module-level pointers.

## Focus verdicts

- Decision 5 and the same-file acceptance criterion: a merged batch can cost less than its cards alone through the per-fork messages paid once per batch; the exact DP can therefore pick a multi-card batch whenever that term times the start-up context exceeds the later cards' added context. Not a defect beyond F6.
- Decisions 1, 3 and 8 on the profile type: see F3.
- Decision 4 against Decision 5: read set and message count are each defined once, in Decision 4, and each coefficient is applied once; the estimate definition is F2.
- Thresholds against the scale factor: see F1.
- Decision 10 against Decision 11 on a state without a partition: settled by Decision 11; wording only, see F4.
