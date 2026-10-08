---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D5 and D1, Acceptance criteria bullet 1
    summary: The seed call is told ClusterExcludes and asked for exclude_lenses that D5 has Go ignore, and D5's settled-lens guidance has no latest round to apply to at seed
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D5 against D1's Bound
    summary: The empty-union fallback to round N-1's lens set is undefined when D1's switch made round N-1 solo or a different fan, and the union may name lenses absent from the current fan
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D3
    summary: D3 says no lens restates the rubric, yet edge-cases, decision-coverage, card-structure and verify-tests paraphrase the rubrics' Also flag bullets, a second copy that drifts when a rubric changes
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D3, acceptance lens
    summary: The acceptance lens asks that must-cover scenarios name a test tier, while the writer stencil has a scenario name a behavior and not a test, which the symmetry rule forbids flagging
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Constraints, PATTERN-review-round bullet
    summary: The constraint says the fixer starts only after Go accepts the review, but under the default fix_start parallel the fixer session starts at once and waits on the ready marker
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Acceptance criteria, usage-record bullet
    summary: D8 has the usage record name the fan, which is what tells the measurement's two arms apart, but no acceptance criterion checks it
  - id: F7
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md D8
    summary: D8's last two sentences restate the PATTERN-shuttle-provider-seam and PATTERN-durable-vs-ephemeral-state constraints already listed under Constraints
---

### [MEDIUM] The seed call writes exclusions that D5 discards

**Location:** `_lyx/discussion/decision-record.md`, D5, D1 and the first Acceptance criterion.

**Issue:** D1 tells the Bouncer row `ClusterExcludes` whenever the segment's fan value is non-empty.
In `internal/shedadapters/bouncer.go`, the seed call and the judge call both copy `focusSchemaMarkers(b.cfg.ClusterExcludes)` into their values, so the seed call that writes the round-1 focus file is asked for a required `exclude_lenses` key.
D5 then has Go ignore whatever the seed call writes there.
The fanned-variant text D5 adds to `focusSchemaMarkers` tells the judge to exclude a lens only when "the latest round's findings from it carry nothing at MEDIUM or worse", and at seed there is no latest round.
So the record asks the seed call for a decision it cannot ground and then silently drops the answer.
The first Acceptance criterion ("their Bouncers ask for `exclude_lenses`") does not say whether that includes the seed call.

**Fix:** Decide in D5 what the seed call sees.
Either the seed call gets the focus-only schema, so round 1 asks for no exclusion, or the fanned variant states that round 1 always runs the whole fan.
State whether Go logs a non-empty round-1 `exclude_lenses` it drops, and make the first Acceptance criterion name which Bouncer call asks for `exclude_lenses`.

### [MEDIUM] D5's union and fallback are undefined across D1's mid-generation fan switch

**Location:** `_lyx/discussion/decision-record.md`, D5, against D1's Bound.

**Issue:** D1's Bound allows a value changed between rounds to "switch a segment between fanned and solo mid-generation".
D5 defines round N's exclusions as the union of `exclude_lenses` over the generation's focus files for rounds 2 to N, and its empty-union fallback as "round N-1's lens set".
Neither definition covers that switch.
If round N-1 ran solo, it has no lens set.
If round N-1 ran a different fan, its lens set names lenses the current fan does not have.
A switch to a different fan name also leaves the union holding lenses the new fan lacks, which `applyClusterExclude` in `internal/burlerengine/profile.go` only warns about.
The "would leave no lens" test is then computed against a fan the earlier exclusions were never chosen for.
D1 makes the switch legal, so this is a missing decision, not a hypothetical.

**Fix:** State in D5 that the union is taken over the round's resolved fan, ignoring names that fan does not have.
State what the fallback is when round N-1 ran solo or another fan, for example the whole current fan.
Alternatively, have a change of fan value restart the exclusion union, the way a new generation does.
Add an Acceptance criterion row for one switch case.

### [MEDIUM] D3's lens texts restate the rubrics that D3 says they never restate

**Location:** `_lyx/discussion/decision-record.md`, D3.

**Issue:** D3's Decision says "none restates or overrides the segment rubric".
Its Rationale says the lenses are drawn from the rubrics' "Also flag" lists, and several lens texts are condensed copies of those bullets.
`card-structure` lists the Plan-Review rubric's Granularity, Card order, Member glyphs, re-sign arrow, `ImpactSummary` and `Custom` bullets.
`verify-tests` paraphrases its Verify coverage and Test economy bullets.
`decision-coverage` paraphrases its Fidelity to the answer key bullet.
`edge-cases`' "every new verb, flag, edge or weakened guard states its bound" restates the attack-surface bullet both rubrics carry.
Those copies live in `burler.yaml`, which is seed-only, so once a rubric changes the lens text keeps the old wording on every seeded hub.
This is the drift `PATTERN-producer-pointer-rule` guards against for instruction files.
The record contradicts itself, and the copies go stale silently.

**Fix:** Phrase each lens as a pointer to the rubric items it emphasizes, naming them, for example "the rubric's card-structure items: granularity, order, glyphs".
Do not paraphrase their rules.
Alternatively, drop the "none restates" claim and accept the copy as a stated maintenance cost in D3.

### [LOW] The acceptance lens steers toward a test-tier finding the symmetry rule forbids

**Location:** `_lyx/discussion/decision-record.md`, D3, the `acceptance` lens.

**Issue:** The lens asks that "must-cover scenarios name behaviors in the lowest test tier their substrate needs".
The writer stencil (`loom-template-discussion.md`, Acceptance criteria rule) says a scenario "names a behavior to cover, not a test to write".
A test tier is a property of a test, which the writer is told not to specify.
The rubric's symmetry note forbids flagging as missing what the writer stencil tells the writer not to gather.
This lens invites exactly that finding in every fanned Discussion round.

**Fix:** Drop the test-tier clause from the `acceptance` lens.
The tier belongs to the plan's `verify-tests` lens, which already carries it.

### [LOW] The review-before-fix constraint misstates when the fixer starts

**Location:** `_lyx/discussion/decision-record.md`, Constraints, the `PATTERN-review-round` bullet.

**Issue:** The bullet says "the fixer starts only after Go accepts the consolidated review".
Under `fix_start: parallel`, the default and the empty value (`internal/burlerengine/doc.go`), the fixer session starts at once beside the reviewer and waits on the ready marker.
The invariant the record needs is that the fixer acts only after the marker.
As written, a plan writer could read the constraint as requiring `after-review`, or flag the default mode as a breach.

**Fix:** Reword the constraint: the fixer acts only after the ready marker, which Go writes once it has accepted the consolidated review, in either `fix_start` mode.

### [LOW] No acceptance criterion checks the fan name in the usage record

**Location:** `_lyx/discussion/decision-record.md`, Acceptance criteria, the usage-record bullet.

**Issue:** D8 says "the round's record also names the fan (empty when solo)".
With D1's mid-generation switch, that field is the only thing in a record that tells a fanned round from a solo one.
The criterion lists the model, wall time, token counts, fork count, lenses and fork share, but not the fan name, so a regression that drops it passes every criterion.

**Fix:** Add the fan name, empty for a solo round, to the usage-record criterion.

### [NIT] D8 restates two listed constraints

**Location:** `_lyx/discussion/decision-record.md`, D8.

**Issue:** "Reading usage is a provider capability in `claudeengine`..." restates the `PATTERN-shuttle-provider-seam` constraint.
"The record is durable: it lives in the run directory..." restates the `PATTERN-durable-vs-ephemeral-state` constraint.
Both constraints are already under `## Constraints`.

**Fix:** Delete the two sentences from D8, or keep only the part a constraint does not carry: an unreadable transcript records unknown tokens and never fails or delays the round.

## Verdicts on the focus directives

- D7's premise holds.
  The burler reviewer half is always gated by `ReviewGateEntry` (`internal/burlerengine/engine.go`, `roundHalves`).
  `expiredTurnEnd` in `internal/shuttleengine/wait.go` returns still-waiting for any outstanding `BackgroundFork`.
  The `claudeengine` background parser reports Stop-payload forks as `BackgroundFork` with `SignalPayload`.
  `TestPollEventsTick_ShellExpiry`'s row "a fork beside payload-reported shells keeps the gated run waiting" and `TestPollEventsTick_ForkOutstandingKeepsWaitingPastBound` already pin it.
  `review_timeout_min` exists in `internal/loomengine/config.go`.
  Not a defect.
- Attack surface: D1, D4, D5 and D7 each state what the change lets through and what bounds it.
  The one gap, D5 across D1's switch, is F2.
- D2 and the Open risks: the operator note on the board entry ("a fanned review's forks run on sonnet, while a solo reviewer runs on opus"; "the fan's fork model is its own setting") matches D2, the `sonnet[high]` assumption, Q3 and the first Open risk.
  D2 changes no reviewer or fixer role.
  The review-before-fix wording is F5.
- D6 and D8: the facts-file lens section carries what D5's settled-area test reads, the per-origin counts by severity, and the excluded lenses.
  The usage record carries per-half tokens and wall time, the fork share and the fan, enough for per-round and per-segment sums in both arms.
  The only content to cut is F7.
- Acceptance criteria against Decisions: every Decision from D1 to D9 has a criterion.
  Apart from the general `go test ./...` line, no criterion lacks a Decision.
  F1 and F6 are the coverage gaps.
