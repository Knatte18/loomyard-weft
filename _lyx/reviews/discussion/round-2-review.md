---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:135
    summary: D9 claims the existing stateChangeReady rules keep noticeRelayWait after the hold ends on a hand-on, but those rules send the notice at once with report none yet, so the double notice returns on the hand-on path
    fix: In D9's "When the hold ends" bullet, replace the claim that the existing rules suffice with a decided change to stateChangeReady in internal/battenshed/notice.go - after an eligible-stop hold ends, the report wait is measured from the hold's end rather than from the episode's since, so a watched awaiting stop with a parent notice waits noticeReportWait from the hand-on for the driver's stop report and then noticeRelayWait from that report; a stop the hold never covered keeps today's rules. Rewrite the acceptance criterion at decision-record.md:185 to pin it - an attempt hands on at a watched awaiting stop with a parent notice more than noticeReportWait after the stop, with no stop report yet, and batten sends no notice until the driver's report plus noticeRelayWait passes without a decision record, or noticeReportWait after the hand-on passes with no report.
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:83
    summary: D4 promises a run_settings refusal naming review_max_bounces, but PrepareRunSettings runs Migrate before ValidateLocal, so the migrated document is refused as setting the guard key skip_review, a key the task never set; the mechanism that yields D4's wording is left open
    fix: In D4, replace "PrepareRunSettings applies the same hook, so a task's run_settings ... is refused" with a decided mechanism that names where the refusal is raised and on which document, for example - darn's run_settings document is checked by D2's shared local-values check before the Migrate hook runs in PrepareRunSettings (internal/configreg/runsettings.go), so review_max_bounces 0 is refused there with the stated wording and the migrated skip_review never reaches configengine.ValidateLocal. Extend the acceptance criterion at decision-record.md:177 to say the refusal does not name skip_review as the key set.
  - id: F3
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:133
    summary: The Go driver's hand-over at a binary-eligible blocked stop writes a stop report and a park marker before the successor driver reaches the tackler or arbiter, so D9's end condition ends the hold and the notice goes ahead of the arbiter
    fix: In D9's end-condition bullet, add that a stop report or park marker written by a hand-over (a settle record with action hand-over at this stop key, later than the stop) does not end the hold, which then runs on to the successor's own end conditions or the backstop; add a criterion that a watched blocked eligible stop whose driver hands over to a fresh driver gets no batten notice before the successor's arbiter hands on.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:130
    summary: D9 takes blocked-stop eligibility from D5's function, but D5 defines that function only for an awaiting stop's escalation cause; the blocked rule (a verify halt naming a target is refused) lives in D6 and the verb
    fix: In D5, state that the one shared Go function answers the verb's whole awaiting and blocked eligibility (D5's cause rule and D6's verify-halt rule) for the verb, the Go driver and batten's hold; or in D9's bullet name both D5's cause rule and D6's blocked rule as what battencli wires.
  - id: F5
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:155
    summary: The Constraints line cites refusal-spec row 356 by number, which the parallel overview-seats edits to refusal-spec.md (Open risks) can shift
    fix: Replace the text row 356 (darn budget below 0) with the row's title, the darn.yaml review_max_bounces below 0 row, quoted in backticks as it appears in refusal-spec.md.
---

### [BLOCKING] D9's post-hold notice does not keep the relay wait it claims

**Location:** `_lyx/discussion/decision-record.md:135`, and the criterion at `:185`.

**Issue:** D9 says that when the hold ends "the notice follows `stateChangeReady`'s existing rules, so an awaiting stop with a parent notice that this batten watches keeps `noticeRelayWait` ... and the driver's own relay after a hand-on is not doubled."
`stateChangeReady` (`internal/battenshed/notice.go:427-452`) for a watched awaiting stop with a parent notice returns `waited` when no stop report written at or after the episode's `since` exists, where `waited` is `now - episode.since >= noticeReportWait`.
On the hand-on path, the driver writes its stop report only after the verb returns the `hand-on` verdict (`shed-template-driver.md`, `## Arbiter`: "write the stop report ... then park and notify the parent").
So the hold ends on the hand-on attempt record before any qualifying report exists, and by then `waited` is long true, since an eligible hold always outlasts `noticeReportWait`.
`stateChangeReady` returns ready at once, batten sends `report none yet`, and the driver's relay follows seconds later: the double notice the task removes, on the main hand-on path.
`noticeRelayWait` is measured from the report's time and is never reached on this path.
The criterion at `:185` pins the same false outcome, so a plan writer implementing "the existing rules" ships the defect while believing the criterion holds.
The wording came in through round 1's F2 fix text, which proposed the same premise, so applying that fix as written does not settle this.

### [MEDIUM] D4's run_settings refusal wording has no mechanism

**Location:** `_lyx/discussion/decision-record.md:83`, criterion at `:177`.

**Issue:** `PrepareRunSettings` (`internal/configreg/runsettings.go:60-73`) runs `module.Migrate`, then `configengine.ValidateLocal`, then `CheckValues`.
With D4's hook, `{"darn": {"review_max_bounces": 0}}` is rewritten to `skip_review: true` with the template budget before validation.
`ValidateLocal` (`internal/configengine/layers.go:178-180`) then refuses with `key "skip_review" is guard and cannot be set in a local override`, which names a key the task did not set and no way forward.
D4 asserts the refusal "names `review_max_bounces` as the key set and `skip_review` in the hub's `darn.yaml` as the way forward" as a consequence ("so") of applying the hook, but that wording does not follow from the hook; the plan writer must invent where the refusal is raised.
Changing the generic order for every module is not free either, since loom's `Migrate` raises keys to their floor and today silently repairs such a document.

### [LOW] A Go driver hand-over ends the hold early

**Location:** `_lyx/discussion/decision-record.md:133`.

**Issue:** `decideDrive` (`internal/loomcli/driverules.go`) places the binary-change hand-over before the tackler and the `ResumeReturned` arbiter rule, so at a `blocked` stop after a deploy the Go driver hands over first.
`handOver` (`internal/loomcli/drive.go`) "leaves the stop report, the park marker" and spawns a successor, which then takes the stop to the tackler and the arbiter.
D9 ends the hold "at once when the driver writes a stop report or park marker in this episode", so batten notifies the parent while the successor's arbiter is still to run.
Deploys follow every landing, so this is not a far-fetched path, though it is narrower than F1.

### [LOW] D9 cites D5's function for blocked eligibility it does not define

**Location:** `_lyx/discussion/decision-record.md:130`, against D5 at `:89-92`.

**Issue:** D5's shared function is defined as answering whether an `awaiting` stop's pending escalation has cause `hard-cap` or `circling`.
D9's hold covers "a `blocked` stop the verb would admit" and says "Eligibility comes from D5's function".
The blocked rule, refusing a Merge-In or Publish verify halt whose failure record names a target, is D6's and lives in `internal/loomcli/arbitereligibility.go`.
The record does not say whether the shared function absorbs it, so batten could hold a verify halt the verb refuses, or carry its own copy of the rule, which D9 forbids.

### [NIT] Refusal-spec row cited by number

**Location:** `_lyx/discussion/decision-record.md:155`.

**Issue:** "row 356 (darn budget below 0)" is correct today (`contracts/specs/refusal-spec.md:356`), but `overview-seats` edits that file in parallel (Open risks), so the number can point at another row by the time the plan is written.

## Focus verdicts

- D9's cause: not a defect.
  The record does not single out one window, but the hold starts from the stop's time and ignores the marker and its liveness, so a plan writer can implement it without finding the cause.
  The three named windows match the code: the verb writes the marker in `recordStartedArbiter` after spawning and removes it in `finish` (`internal/loomcli/arbiter.go`), and `arbiterInFlightFrom` (`internal/battencli/wire.go:115-133`) reads a marker whose GUID has no live row as not alive, which `arbiterHolds` does not hold on.
- D2 and D1: not a defect.
  `loomengine.ValidateLocalValues` checks `driver_kind`, `tackler_timeout_min` and `tackler_max_spawns` only (`internal/loomengine/config.go:744-770`), as D2 states.
  D2 decides that loom gains the upsert check, and the criteria at `:179-180` agree.
- D5: not a defect.
  An unreadable record or unknown cause is not eligible with a reason.
  With `EscalationPending` false, `decideDrive` falls through to the relay-and-hand-back rule for an awaiting stop with a parent notice (`internal/loomcli/driverules.go`), the verb's awaiting branch refuses with `arbiterHandBackWayForward`, and the hold does not start, so all three consumers agree.
  The Go driver's `facts` no longer errors on this case once the shared function stops returning an error, which D5 states.
- D4 both keys: not a defect; decided at `:79` and pinned at `:176`.
  The `run_settings` half of D4 is F2.
- Criteria against D9: F1. Start conditions at `:186` and the restart case at `:185` agree with D9.
