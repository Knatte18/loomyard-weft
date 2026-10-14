---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md, D9, bullet "Plan finds the cause behind the notice in both incidents"
    summary: D9 defers the double-notice root cause to Plan, names no incident the plan writer can find, and rests the pre-marker hold on an unverified cause
    fix: >-
      Settle the cause in Discussion and replace the bullet with a Decision.
      Read the two candidates in the code now.
      First, the pre-marker window, where the LLM driver takes longer than noticeReportWait (3 minutes, internal/battenshed/notice.go) to run the arbiter command, given that the verb writes the marker right after Shuttle.Start (internal/loomcli/arbiter.go, recordStartedArbiter).
      Second, a marker that exists but reads not alive, since arbiterInFlightFrom (internal/battencli/wire.go) reports Alive only when a live row of the pair's reed directory carries the marker's strand guid, so arbiterHolds holds nothing.
      State in D9 which cause the hold fixes and, if the second is possible, the change that fixes it.
      If the evidence cannot settle it, drop the bullet and add an Open risk that names the presumed cause and the evidence, the logging-cleanup notice at 2026-10-13T09:26:22Z and the tender notice at 2026-10-13T15:32:29Z in the hub's .lyx/notices.log, so the record carries what the plan writer needs without reading the support log.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md, Acceptance criteria, the criterion beginning "Batten sends no notice while a tackler escalates"
    summary: The criterion's hold-ending cases do not match D9, and its sends one at once contradicts the relay wait batten keeps for a watched awaiting stop
    fix: >-
      Rewrite the criterion so its cases match D9 exactly.
      D9 ends the hold on a driver stop report, a park marker, or a hand-on attempt for the stop key, and does not apply when the arbiter is off, its spawns are spent, or a resume at this stop key was applied.
      Add the park marker case the criterion lacks.
      Split the off, spent and applied-resume cases into a criterion saying the hold does not start for them.
      Replace sends one at once with the hold ends at once and the notice then follows stateChangeReady's existing rules, which for a watched awaiting stop with a parent notice keep the noticeRelayWait and decision-record check, so the driver's own relay after a hand-on is not doubled.
      In D9, state whether spawns becoming spent or the arbiter being switched off mid-hold ends a hold already running.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md, D2 and the Acceptance criterion "Loom's existing config behaviour and refusals are unchanged"
    summary: D2 rests on a false premise, since loom's ValidateLocalValues checks only driver_kind and the tackler knobs, so whether loom gains an upsert-time budget check under D1 is undecided
    fix: >-
      In D2, replace as loom's ValidateLocalValues does with a decision.
      Recommended is that internal/reviewbudget exposes one local-values check of the budget key family, and both loom's and darn's CheckValues hooks call it, so the two recipes cannot drift at upsert either.
      Then amend the loom-unchanged criterion to name the one change, that a loom run_settings budget below its bound is now refused at lyx board upsert, and add a criterion for it.
      If loom is instead left out, say in D2 that only darn checks budget values at upsert and why.
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md, D5
    summary: D5 claims the verb, the Go driver and batten cannot disagree, yet leaves the shared function's error contract open, so an unreadable or unknown-cause record routes three ways
    fix: >-
      State in D5 that the shared function returns not eligible with a reason for an unreadable record or an unknown cause, never an error.
      Today the verb turns such a read error into a refusal that the LLM driver relays as the planned hand-off (D8).
      The Go driver's facts read returns the error and hands back with the driver could not read the facts of the stop and no relay (internal/loomcli/drive.go, the d.facts call).
      Batten's seam error holds nothing.
      With not eligible, the Go driver takes its relay-and-hand-back rule for an awaiting stop with a parent notice, as the LLM path does.
      Add an Acceptance criterion that the Go driver relays and hands back at an awaiting stop whose escalation record is unreadable.
  - id: F5
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md, D4 and the Acceptance criterion on the hub darn.yaml migration
    summary: D4 does not say what the migration does to a document holding review_max_bounces 0 and skip_review together, and the criterion's migrates without conflict pins no outcome
    fix: >-
      Decide in D4 which key wins when a document holds review_max_bounces 0 beside an explicit skip_review, and pin the result in the criterion.
      Recommended is that an explicit skip_review is kept as written, review_max_bounces is raised to the template default, and the migration line names both keys, since an explicit skip_review is the newer statement of intent.
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md, D4
    summary: D4's migration adds a key with a new meaning, outside the Migrate contract configreg documents for its hook
    fix: >-
      Add to D4 that the Migrate field's doc comment on configreg.Module (internal/configreg/configreg.go), which today allows only rewriting keys the module declares retired or raising a key to its floor, widens to admit rewriting a retired meaning of a key into its replacement key.
  - id: F7
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md, Scope In
    summary: D5's shared function needs a pending escalation's cause, which no exported shedadapters reader returns, yet internal/shedadapters is missing from Scope In
    fix: >-
      Add internal/shedadapters to Scope In, for an exported reader of the pending escalation's cause.
      shedadapters.PendingEscalation returns only the round, the brief path and whether one is pending, and readEscalation, which returns the cause, is unexported (internal/shedadapters/escalation.go).
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md, D3, and Scope In listing contracts/stencils/orch/orch-template-role.md
    summary: The orch stencil is in Scope, but no Decision changes it, only the optional Notes say it names review_max_bounces 0 as off
    fix: >-
      Add to D3 that the orch stencil's run-loop description replaces darn's review_max_bounces 0 as off with skip_review in the hub's darn.yaml.
      The Notes subsection is optional and non-binding, so it alone does not carry the requirement.
  - id: F9
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md, D6 Decision
    summary: D6 cites commit 30385e1ae as history, which no plan writer requirement depends on
    fix: >-
      Delete already refused since 30385e1ae and keep the rule itself.
      The behaviour D6 keeps is stated by the verb's existing refusal.
---

### [BLOCKING] D9 defers its root cause to Plan, which cannot act on it

**Location:** `_lyx/discussion/decision-record.md`, D9.

**Issue:** The bullet "Plan finds the cause behind the notice in both incidents and fixes it, whether or not it is the pre-marker window" is a deferral, not a decision.
The writer's stencil says to design the full scope now and never defer.
The decision record never says which two incidents it means.
They are named only in the support log's interview turn 4 (`logging-cleanup` at 09:26:22, `tender` at 15:32:29), and `Plan-Write` never reads the support log, so the completeness-before-leanness test fails: a requirement depends on content the plan writer cannot see.

The hold itself rests on the presumed cause, and the code leaves two causes open.
`recordStartedArbiter` writes the marker straight after `Shuttle.Start`, so the verb's own pre-marker window is short.
A long window needs the LLM driver to wait more than three minutes before it runs the arbiter command.
`arbiterInFlightFrom` reports `Alive` only when a live row of the pair's reed directory carries the marker's strand guid, and `arbiterHolds` holds only an alive arbiter.
A marker that exists but reads not alive lets the notice go after `noticeReportWait`.
The pre-marker hold does not catch that case, because D9 hands over to the marker hold as soon as the marker exists.
Both incident notices went out at about 3.5 minutes with `report none yet`, which fits either cause.

### [MEDIUM] The Acceptance criteria's hold-ending cases differ from D9's

**Location:** `_lyx/discussion/decision-record.md`, Acceptance criteria.

**Issue:** D9's end conditions are a driver stop report, a park marker, or a hand-on attempt for the stop key.
Its preconditions are the arbiter on, spawns left, and no applied resume.
The criterion lists "a hand-on, a driver stop report, an off or spent arbiter, or an applied resume", leaves out the park marker, and turns the preconditions into end events.
It also says batten "sends one at once".
`stateChangeReady` sends a watched awaiting stop's notice with a parent notice only after `noticeRelayWait` with no decision record, which is what keeps batten from doubling the driver's own relay.
Read literally, the criterion requires a second notice right after the driver relays a hand-on, the double notice this task removes.

### [MEDIUM] D2's upsert check copies a loom check that does not exist

**Location:** `_lyx/discussion/decision-record.md`, D2.

**Issue:** `loomengine.ValidateLocalValues` checks `driver_kind`, `tackler_timeout_min` and `tackler_max_spawns` only.
Loom refuses `review_max_bounces` below 1 only at `LoadConfig`, and the `refusal-spec.md` rows for loom's review keys are load-time halts.
D1 puts the value checks of the key family into `internal/reviewbudget` for both modules.
So D2's "as loom's does" either adds a new upsert refusal to loom, which contradicts the loom-unchanged criterion, or gives darn a check loom lacks, which reopens the drift D1 exists to prevent.
The record does not choose between these.

### [MEDIUM] D5's single function has no stated contract for an unreadable record

**Location:** `_lyx/discussion/decision-record.md`, D5.

**Issue:** The three consumers handle a read error differently today.
The verb refuses, and the LLM driver relays the `parent_notice` (D8).
The Go driver's facts read returns the error and hands back as "could not read the facts of the stop", without the relay.
Batten's seams warn and hold nothing.
D5 says that one function makes them agree, but agreement depends on that function reporting an unreadable record or an unknown cause as not eligible rather than as an error, and D5 does not say so.
The criteria pin the Go driver only at a `budget` escalation.

### [MEDIUM] D4 leaves the both-keys migration open

**Location:** `_lyx/discussion/decision-record.md`, D4 and its migration criterion.

**Issue:** D4 rewrites a document holding `review_max_bounces: 0` to `skip_review: true` but does not say what happens when an explicit `skip_review: false` sits beside it.
The criterion's "migrates without conflict" names no resulting value.

### [LOW] D4's migration sits outside the documented Migrate contract

**Location:** `_lyx/discussion/decision-record.md`, D4.

**Issue:** `configreg.Module.Migrate`'s doc comment says the hook "changes only the keys its module declares retired or raises a key to its module's floor".
D4 adds `skip_review` and turns one key's value into another key's meaning, and the record does not say that the contract widens.

### [LOW] internal/shedadapters is missing from Scope

**Location:** `_lyx/discussion/decision-record.md`, Scope In.

**Issue:** D5 needs the pending escalation's cause.
`shedadapters.PendingEscalation` drops it, and only the unexported `readEscalation` returns it, so the package changes.
Scope In does not list it.

### [LOW] The orch stencil's change traces only to the optional Notes

**Location:** `_lyx/discussion/decision-record.md`, D3 and Scope In.

**Issue:** Scope In lists `orch-template-role.md`.
D9 keeps the orch stencil's circling guidance as it is.
The only change, retiring "darn's `review_max_bounces` 0 as off", appears only in the optional Notes subsection, which binds nothing.

### [NIT] D6 carries commit history

**Location:** `_lyx/discussion/decision-record.md`, D6.

**Issue:** "already refused since `30385e1ae`" records when the refusal landed, which no plan requirement depends on.

## Rejected

- D9's bound: the pre-marker hold states its end conditions, and its backstop (the stop's status-file time plus `arbiter_timeout_min` plus the report wait) reaches the parent if the driver dies or never spawns. Batten's episode `since` already comes from the status file's mtime at a batten start, so the restart claim holds. Not a defect, apart from F1 and F2.
- D5/D8 routing of an awaiting stop with no escalation and of PR-Gate: the verb refuses (no pending escalation), the Go driver's `decideDrive` relays and hands back on a non-empty `parent_notice`, and batten's hold covers hard-cap and circling only. The three consumers agree. Only the unreadable-record case differs (F4).
- D2 and D3 bounds: D2 states what a run's `run_settings` can now lower and names the Bouncer's soft-cap rule as the remaining guard, which matches `softCapRule` in `internal/shedadapters/bouncer.go`. D3 states that the switch skips the only review and keeps it a hub-only guard key. Both are bounded.
- Scope coverage of D1's loom-side move: Scope In names `internal/loomengine`'s cap type, resolution and cap route moving into the new package. Present.
- D8's "This replaces the board entry's narrowing" and D5's judge-cap-call rationale were kept, not flagged for relocation: each tells the plan writer which board-entry instruction not to follow, or why `budget` is no longer the arbiter's.
