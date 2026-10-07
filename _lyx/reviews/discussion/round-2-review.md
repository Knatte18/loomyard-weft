---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:57
    summary: Decision 5 cost model has no term that makes merging cheaper, so the DP always returns one-card batches and the acceptance criterion that two small cards sharing a file group is unsatisfiable
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:34
    summary: Decision 1 puts the calibration report before the profiles, but Decision 3 needs a batcher.yaml profile (loader and template coefficients) that Decision 8 creates later
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:50
    summary: Decision 4 calls the estimator a pure function of cards and a size source, while its outputs need Decision 5's coefficients, and both Decisions define the read set and message count
  - id: F4
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:62
    summary: The unit of alone_above and budget is not fixed against the calibration scale factor, and the calibration rule sets no threshold
  - id: F5
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:102
    summary: Decision 11 does not say what a rebaseline does on a state with no recorded partition, where Decision 10 pins every verb to identity
  - id: F6
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:143
    summary: A poor fit has no criterion and stays inactive restates the default, so the late card has no rule for when to commit weights
  - id: F7
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:44
    summary: Decision 3 asserts every past card has one measured one-card fork cost but does not say what the table holds for a card with several forks or none
  - id: F8
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:83
    summary: Decision 8 lists an invalid parameter as a load error without saying what is invalid, though the DP and the cost model assume non-negative coefficients and max_cards of at least 2
  - id: F9
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:106
    summary: Decision 11 says cards added, removed or reordered in the tail are accepted, but the existing changed-file guard still refuses any such plan file not named with --card
  - id: F10
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:52
    summary: Decision 4 gathers signals Decision 5 never uses and leaves the test package entry and a multi-directory card's package entries undefined
  - id: F11
    severity: NIT
    class: consistency
    location: _lyx/discussion/support-log.md:14
    summary: Interview item 4 cites Decision 12 for retiring the Batchable column, which is Decision 13
---

### [BLOCKING] The cost model never rewards merging

**Location:** `_lyx/discussion/decision-record.md:57` (Decision 5), `:64` (its rationale), `:164` (acceptance criterion)

**Issue:**
Decision 5 charges card k's messages the start-up context plus the union of the read sets of cards 1..k, per message, and a card's message count depends on that card alone.
Alone, card k's messages carry start-up plus R_k.
In a segment they carry start-up plus the union U_k, and U_k always contains R_k, so every merged card costs at least what it costs alone ("a later card pays for an earlier card's accumulated context" says as much).
Shared files do not help, because the card pays for its own read set either way; they only make the merged cost equal to the singles' sum, never lower.
The start-up context is charged per message, so it saves nothing either.
No parameter in the coefficient list (start-up context, messages per target, per test file, per `Uses` entry, context per line, package weight) is paid once per fork.
So the exact DP of Decision 7 returns the all-singletons split on every input, and its tie rule ("more, smaller batches") sends even the equal-cost cases the same way.
The rationale line "merging pays only when cards share much and both are small, and the model says when" is false as written.
The acceptance criterion "Two small cards editing the same file group under a cost profile" can never hold, and the Goal (cut tokens by running related cards in one fork) has no mechanism.
Calibration cannot repair this: every past fork was a one-card fork, so a per-fork overhead is confounded with the per-card terms there, and the single scale factor of the Auto-mode rule has no intercept to fit it.

**Fix:**
Add a per-fork term to Decision 5: a fixed number of messages a fork spends before its first card, each carrying the start-up context, paid once per batch; add it to the profile parameters and the weights list.
State in Decision 5 that the saving is that term amortised over the batch, and that a later card's accumulated context is the price.
Note in Open risks that the per-fork term cannot be identified from past one-card forks, so it ships at its initial value and the saving is only measured on real runs.
Keep the acceptance criterion; it now follows from the model.

### [MEDIUM] Calibration is ordered before the profiles it needs

**Location:** `_lyx/discussion/decision-record.md:34` (Decision 1), `:47` (Decision 3), `:82` (Decision 8)

**Issue:**
Decision 1 puts the calibration report third, before "the cost batchifier, the profiles and every webster change".
Decision 3 has the report run with "the coefficients of a profile in `batcher.yaml` that the caller names", and the Auto-mode assumptions have it run on the template's initial cost-profile coefficients.
Parsing a `profiles:` block and the template's cautious `cost` profile are Decision 8's work, which sits after the report.
The report's `Uses` of the estimator is satisfiable, since the estimator card is earlier, but Plan-Gate's `uses-later-target` check matches exact entries only, so it will not catch the profile-loading dependency, and a plan that follows Decision 1 either builds a second coefficient loader in the report or reaches forward in the card order.

**Fix:**
Say in Decision 1 that the profile type, its `batcher.yaml` parse and the template's initial cost profile land in the estimator's card (or a card before the report), and that the later profiles card adds the registry constructor, `active:` resolution and the load-error rules.
Move the matching sentence out of Decision 8's list so each part is owned once.

### [MEDIUM] Estimator inputs and outputs are defined twice and disagree

**Location:** `_lyx/discussion/decision-record.md:50` (Decision 4), `:57` (Decision 5)

**Issue:**
Decision 4 titles the estimator "a pure function of cards and a size source" and gives it two outputs, a message count and a read set of "weighted size" entries.
Decision 5 says the message count is "messages per target, per test file and per `Uses` entry, times the matching coefficients", and the weights are line count times a context-per-line coefficient, so neither output exists without the coefficients.
Decision 3 confirms the estimator takes coefficients ("using the coefficients of a profile").
Decision 5 then says a segment cost is computed "from those two per-card outputs of Decision 4 and the profile's coefficients", which reads as weighting twice.
Read set and message count are each defined in both Decisions, in different words.

**Fix:**
Define the estimator once in Decision 4: its inputs are the cards, a size source and the profile coefficients, and its outputs are the per-card message count and weighted read set.
Decision 5 then only adds the segment sum over those outputs and the start-up context.
Delete the second definition of the read set and the message count from Decision 5.

### [MEDIUM] Unit of the thresholds against the scale factor

**Location:** `_lyx/discussion/decision-record.md:62` (Decision 5), `:68` (Decision 6), `:142` (Auto-mode assumptions)

**Issue:**
Decision 5 puts the cost in "the model's own context-token unit" and says the calibration report scales it to tokencount's weight, which is input plus 1.25 times cache writes plus 0.1 times cache reads plus 5 times output (`tools/tokencount/main.go`), a different quantity from context read.
Decision 6 compares `alone_above` and `budget` against cost under Decision 5 and justifies `alone_above` by what "cost a fork hundreds of thousands of tokens", which is a measured quantity.
The Auto-mode rule then applies one scale factor "to the model's unit" and commits the numbers in the template.
The record does not say which coefficients carry the factor (a factor on cost scales the context-unit coefficients, not the message-count ones), whether the thresholds are in the model's unit or the weight unit after scaling, or who sets `alone_above` and `budget` once the unit changes; the rule sets only weights, while the template ships "a low budget".

**Fix:**
State that cost is in tokencount's weight unit after the calibration card runs, that the factor multiplies the context-unit coefficients (start-up context, context per line, package weight), and that the same card sets `alone_above` and `budget` in that unit, with the initial values stated in the model's unit before it.

### [MEDIUM] Rebaseline on a state without a recorded partition

**Location:** `_lyx/discussion/decision-record.md:102` (Decision 11), `:99` (Decision 10)

**Issue:**
Decision 10 pins a state without a recorded partition to the identity batchifier "whatever profile is active", so the fallback reproduces what the run started under.
Decision 11 re-batches the tail "by the then-active profile" and "the recorded partition is replaced", which assumes one exists.
On a legacy state the rebaseline either records a partition and moves the tail to the active cost profile after identity-sized begun batches, contradicting the Decision 10 bound, or keeps identity and records nothing.
The record leaves this open; the in-flight-run guarantee is exactly the case it covers.

**Fix:**
Add to Decision 11 that on a state without a recorded partition the rebaseline keeps the identity batchifier for the tail and records no partition, so the run stays on identity until it ends.

### [LOW] No criterion for a poor fit

**Location:** `_lyx/discussion/decision-record.md:143` (Auto-mode assumptions)

**Issue:**
"If it cannot be produced or the fit is poor, the initial coefficients stay and the profile stays inactive" names no measure or threshold for poor, though the report's summary gives a fit per run and overall.
"Stays inactive" restates the default, since the profile ships inactive in every case, so the sentence says nothing about what the late card then commits.

**Fix:**
State the criterion in the fit summary the report writes (for example the overall median ratio and its spread, with a spread above which the card commits nothing) and say that on a failed or poor fit the template keeps the initial coefficients and the card commits no weights.

### [LOW] Measured weight per card with several forks or none

**Location:** `_lyx/discussion/decision-record.md:47` (Decision 3 rationale)

**Issue:**
The rationale says "every past card has a measured one-card fork cost", but Decision 2 itself lists forks that name no card as unattributed, and a card can be named by more than one fork or by none (a skipped batch, a re-run).
The per-card table has no stated rule for such a card.

**Fix:**
Say in Decision 3 that a card named by several forks takes their summed weight, and a card named by none is listed as skipped with that reason; drop "every" from the rationale.

### [LOW] "Invalid parameter" is undefined

**Location:** `_lyx/discussion/decision-record.md:83` (Decision 8)

**Issue:**
No threshold combination makes the DP infeasible, because a one-card batch is always feasible (Decision 6), and an `alone_above` above `budget` is accepted.
But Decision 8 only says a "missing or invalid parameter" is a load error.
The merged cost is only at least the singles' cost, and the DP is only meaningful, when the coefficients are non-negative; a `max_cards` below 2 is the identity batchifier under another name.

**Fix:**
Define invalid in Decision 8: a negative coefficient, a non-positive `budget` or `alone_above`, or a `max_cards` below 2 for a `cost` profile.

### [LOW] Decision 11 against the existing `--card` naming guard

**Location:** `_lyx/discussion/decision-record.md:106` (Decision 11)

**Issue:**
`Rebaseline` refuses any changed plan file, including an added or removed card file, that is not named with `--card NN` ("changed but not named"; `changedPlanFiles` lists absent files too).
Decision 11 says cards added, removed or reordered after the last begun batch "are accepted", without the naming requirement, so a plan writer could implement acceptance without it.

**Fix:**
Add "each named with `--card NN` as today" to that sentence.

### [LOW] Signals gathered but unused, and the package entries undefined

**Location:** `_lyx/discussion/decision-record.md:52` (Decision 4), `:60` (Decision 5)

**Issue:**
Decision 4 lists line counts of test files, the files of a package directory, and the created, edited and deleted distinction as signals.
Decision 5 uses none of them: the package entry carries a flat weight, test files only add messages, and a created file has no line count.
Both Decisions name "the test package" as an entry without defining it, though Go tests live in the package directory, and neither says what a card with targets in several directories contributes.

**Fix:**
Cut the unused signals from Decision 4 and the size source's package-listing method, or state where Decision 5 uses them; define the test entry as the directory's test files and say a card contributes one package entry and one test entry per distinct directory of its targets.

### [NIT] Stale Decision number in the interview

**Location:** `_lyx/discussion/support-log.md:14`

**Issue:**
Interview item 4 ends "(Decision 12)", but the `Batchable?` column is Decision 13 and Decision 12 is the execution order.

**Fix:**
Change it to Decision 13.
