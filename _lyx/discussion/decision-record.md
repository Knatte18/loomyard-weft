# Perch budgets and stops: darn budget per run like loom, arbiter only when needed

## Goal

Darn and loom read their review budget and off-switch keys through one shared config shape, so a darn run takes a per-run review budget through `run_settings` exactly as a loom run does, and switching Darn-Review off is its own guard key.
The arbiter spawns only at stops that need an opus judgement (D5–D8).
The run's parent hears about an arbiter-eligible stop only once the arbiter has handed on, or when no arbiter will settle it (D9).

## Scope

In:

- `internal/darnengine` config: categories, a new `skip_review` guard key, value checks, a config migration, template comments, `doc.go`.
- A new shared package for the review budget key family, used by both `internal/loomengine` and `internal/darnengine` config; loom's cap type, resolution and cap route move into it.
- `internal/loomrecipe` darn routing, `internal/loomcli` darn wiring, cap routes and circling, `internal/configreg` (darn's migrate and value-check hooks, and `PrepareRunSettings`'s validate-then-migrate order).
- `internal/loomcli` arbiter eligibility and the Go driver's arbiter rule.
- `internal/shedadapters`: an exported reader of a pending escalation's cause.
- `internal/battenshed` / `internal/battencli` state-change notice hold for an arbiter-eligible stop.
- Stencils: `contracts/stencils/shed/shed-template-driver.md`, `contracts/stencils/loom/loom-template-arbiter.md`, `contracts/stencils/orch/orch-template-role.md`.
- `contracts/specs/refusal-spec.md` rows, `internal/loomcli/doc.go`, `internal/darnengine/doc.go`, `internal/battenshed/doc.go`, and the new package's row in `pattern/PATTERN-leaf-packages.md`.

Out:

- The board entry's "Later: batten stops" and "general helper" sections: the entry marks both as the operator's open thinking, not asks.
- A darn auto-accept key, and a run-wide skip switch for loom's review perches: not asked.
- A recipe edge for the Merge-In verify-failure goto: batten's one retry and the Go driver's verify-goto rule already perform it, and the arbiter verb refuses that stop (the entry's first open point).
- `lyx report convergence` / `report-timing` counting: owned by `report-timing`; the settle records and arbiter attempt records stay the source it reads.

## Decisions

### D1 — One shared review budget config shape in a new leaf package

**Decision:** A new package, `internal/reviewbudget`, declares the review budget key family once and both loom and darn config use it:

- run-wide keys `review_max_bounces` (category none, at least 1) and `review_hard_max_bounces` (category none, at least 1, not below the resolved soft cap);
- per-perch keys `<perch>_max_bounces` and `<perch>_hard_max_bounces` (category none, at least 0, where 0 takes the run-wide value);
- `<perch>_auto_accept` and a skip key (category guard).

It owns the categories of those keys, their value checks and refusal wording, the soft/hard cap resolution and the cap type (today `loomengine.PerchCaps` and `Config.Caps`), and the layer-aware way forward (today `loomengine.CapRoute`), parameterised by the module's spec so darn's hub-wide stack routes correctly.
`loomengine` and `darnengine` build their `Categories` entries and run their cap checks through it, and restate neither; each module's `ConfigSpec().Categories` still lists every template key, so `LocalKeys`, `LocalDenied` and the every-key-classified test keep working unchanged.
The package imports only the standard library and `internal/configengine`, and `pattern/PATTERN-leaf-packages.md` gains its row.

**Rationale:** The board entry asks that the two recipes cannot drift apart again; one declarer prevents a category or bound mismatch.
It sits below both engines because `darnengine` must not import `loomengine`.
The name avoids "perch", a conversational shorthand CLAUDE.md keeps out of new code names.

### D2 — Darn's budget keys become ordinary keys, minimum 1

**Decision:** Darn's `review_max_bounces` and `review_hard_max_bounces` take category none through D1, so a darn run's local layer, and so its board `run_settings`, can set them.
`review_max_bounces` is refused below 1, as loom's is.
`darnengine.LoadConfig` validates the resolved stack.
`internal/reviewbudget` exposes one local-values check of the key family, and both loom's and darn's `CheckValues` hooks in `configreg` call it, so a `run_settings` budget value below its bound is refused at `lyx board upsert` for either recipe; loom's hook keeps its own `driver_kind` and tackler checks beside it.
This is the one change to loom's behaviour: a loom budget below its bound, refused today only at load, is now also refused at upsert.
Every darn cap refusal names the way forward for the layer its value came from through D1's route: the local layer and the task's `run_settings`, or the hub file from the prime.

**Bound:** a run's `run_settings` can now lower Darn-Review to one round with a hard cap of 1.
From the soft cap on, the Bouncer's rule accepts unconverged when no open BLOCKING or MEDIUM finding is tagged `design-wrong`, and escalates `hard-cap` when one is; that rule, not the budget, is what guards the review, as for loom's per-run budget.
A per-run "no review" stays impossible: the least is one round, and skipping is a hub edit (D3).

**Rationale:** The board entry's direction.
Darn stays hub-wide (`verify` is executable); `PATTERN-hub-wide-config` already lets a hub-wide module read the worktree's local override restricted to its category-none keys, so no layer rule changes.

### D3 — `skip_review` is darn's off switch, a guard key

**Decision:** `darn.yaml` gains `skip_review: false`, category guard.
`true` routes Darn-Write's Done straight to Merge-In, so neither Darn-Review nor Describe runs, the behaviour `review_max_bounces: 0` has today; Merge-In and the parent lock still run.
`review_max_bounces: 0` no longer means off.
The darn routing projection takes the switch beside the budget, and the lightweight wiring that reads the budget for status reads the switch too; there, an unreadable `darn.yaml` degrades to the template's `false` with a warning, as the budget already does, and affects only what status reports.
The orch stencil's run-loop description replaces darn's `review_max_bounces` 0 as off with `skip_review` in the hub's `darn.yaml`.

**Bound:** the switch skips the only review a darn run has, and as a guard it is settable only in the hub's `darn.yaml`, from the prime; a `.local` override or `run_settings` naming it is refused by the existing local-layer check.

**Rationale:** The board entry: the guard stays on the off switch, not on the budget.
The name follows darn's run-wide `review_*` keys; loom's `skip_merge_review` is its per-perch counterpart.

### D4 — A config migration turns an old off budget into the switch

**Decision:** Darn registers a `Migrate` hook in `configreg`: a document holding `review_max_bounces: 0` is rewritten to `skip_review: true` with `review_max_bounces` at the template default; a document without it is unchanged, and running the hook twice changes nothing more.
A document holding `review_max_bounces: 0` beside an explicit `skip_review` keeps `skip_review` as written, since it is the newer statement of intent, gets `review_max_bounces` at the template default, and its migration line names both keys.
The `Migrate` field's doc comment on `configreg.Module`, which allows only rewriting keys the module declares retired or raising a key to its floor, widens to admit rewriting a key's retired meaning into its replacement key.
The hub file is rewritten by `hubreconcile.Ensure` after the binary change, the one reconcile route `PATTERN-binary-change-reconcile` allows; `darnengine.LoadConfig` does not migrate.
A darn load that still finds `review_max_bounces: 0` is refused below 1, and that refusal names `skip_review` from the prime as the way to switch the review off.
`PrepareRunSettings` (`internal/configreg/runsettings.go`) changes its order for every module to validate-then-migrate: it runs `configengine.ValidateLocal` and the module's `CheckValues` on the document as the task wrote it, then the `Migrate` hook on the accepted document.
So a task's `run_settings` of `{"darn": {"review_max_bounces": 0}}` is refused by D2's shared local-values check, through darn's `CheckValues` hook, before the hook rewrites it, and the migrated `skip_review` never reaches `ValidateLocal`; the refusal names `review_max_bounces` as the key set and `skip_review` in the hub's `darn.yaml` as the way forward, with a new refusal-spec row and a test.
Loom's `Migrate` raises only `step_idle_timeout_min` to its floor, a key neither `ValidateLocal` nor any `CheckValues` refuses by value, so a loom document it repairs today is still repaired silently.

**Rationale:** A hub whose `darn.yaml` says 0 today keeps its meaning after the binary change.

### D5 — The arbiter's review triggers narrow to `hard-cap` and `circling`

**Decision:** An `awaiting` stop is arbiter-eligible only when its Bouncer row's pending escalation has cause `hard-cap` or `circling`.
A pending escalation with cause `budget`, from any source (unreadable round inputs, a re-escalated round, a run left awaiting since before `judge-cap-call`), and one whose record is unreadable or carries an unknown cause, is refused like an awaiting stop with no escalation and goes to the parent.
One Go function answers the verb's whole `awaiting` and `blocked` eligibility, this cause rule and D6's verify-halt rule, for the verb, the Go driver's `EscalationPending` fact and batten's hold (D9), so the three cannot disagree.
For an unreadable escalation record or an unknown cause it returns not eligible with a reason, never an error, so the verb refuses with the hand-back way forward, the Go driver takes its relay-and-hand-back rule for an awaiting stop with a parent notice, as the LLM driver does on the verb's refusal (D8), and batten's hold does not start.
The arbiter stencil drops its `budget` rule.

**Bound:** a `budget` escalation now always costs the parent a turn.

**Rationale:** The board entry's case 1: the hard cap and a returning blocker are the arbiter's only review triggers.
Since `judge-cap-call`, the soft-cap call is the Bouncer rule's in `internal/shedadapters/bouncer.go`, so a `budget` escalation is no longer the soft-cap judgement the arbiter was built for.

### D6 — `blocked` and error eligibility stay as they are, case 3 included

**Decision:** The verb's `blocked` rule is unchanged: every `blocked` stop is eligible except a Merge-In or Publish verify halt whose own failure record names a target.
Error eligibility is unchanged, and an error stop reaches the verb only after the driver's own repair escalated (D7): the board entry's case 3 keeps its arbiter spawn.

**Rationale:** The verify-failure record is the only structured way forward Go writes for a halt, and the drivers never match reason text.
The fork or the tackler is the cheap filter in front of case 3, and an arbiter `resume` there saves an orch turn.

### D7 — The LLM driver stencil sends a twice-failed transient to its fork first

**Decision:** In `shed-template-driver.md`, a transient stop that failed twice spawns the one repair fork under the repair cap, like a `producer` error, and reaches `## Arbiter` only when the fork's action is an escalation.
The `## Arbiter` list drops its separate transient line, and the binary-watch eligibility line follows.

**Rationale:** The board table's transient row; the Go driver already sends a transient error to the tackler before the arbiter.

### D8 — The verb is the one eligibility filter; a refusal routes to the stop's own handling

**Decision:** This replaces the board entry's narrowing of the stencil's `parent_notice` line.
The driver stencil keeps running the arbiter command at a `blocked` stop, an `awaiting` stop with a non-empty `parent_notice` (PR-Gate's included), and an escalated error stop, and states that the verb decides eligibility.
A refusal at an `awaiting` stop is the planned hand-off: the driver relays the `parent_notice` verbatim and parks, with the refusal in the stop report but no claim of failure; a refusal at another stop takes that stop's existing handling.
A refused call spawns nothing.

**Rationale:** The LLM driver reads only the envelope, which does not say whether a `parent_notice` comes from a Bouncer escalation or from PR-Gate, so the verb is the one place that can decide.

### D9 — The parent hears about an arbiter-eligible stop only after the arbiter

**Decision:** At an arbiter-eligible stop the parent is notified only on the arbiter's `hand-on`, a refusal, a timeout or a malformed verdict.

- The LLM driver stencil's awaiting line says the `parent_notice` is relayed only on those outcomes, never after a `resume`.
- Batten gains an eligible-stop hold: for an awaiting hard-cap or circling escalation, or a `blocked` stop the verb would admit, with the arbiter on, no applied resume at this stop key, and either spawns left or an attempt at this stop key, it holds its state-change notice from the stop's own time in the status file.
  Eligibility comes from D5's function, which carries both D5's cause rule and D6's `blocked` rule, wired by `battencli`; `battenshed` holds no copy.
- Whether the arbiter marker exists, or its strand reads alive, neither starts nor ends this hold.
  So it covers each window in which an incident notice could have gone: before the driver writes the marker, while a marker's strand reads not alive in the pair's reed directory, and after the verb removes the marker but before the driver acts on the verdict.
- The hold ends at once when the driver writes its park marker in this episode, an attempt for the stop key whose envelope was written in this episode hands on, a resume at this stop key is applied, or the arbiter is switched off.
  An attempt hands on when its saved envelope (`envelope.json`, written by the verb's `finish` for every outcome) carries verdict `hand-on`; `verdict.yaml` is the agent's own word and is absent or overruled for the verb's own hand-ons (timeout, died, missing, unparseable, failed resume precondition).
  The park marker is the one record batten's `StopReport` seam reads (`readStopReport` in `internal/battencli/wire.go`); the stop report files the Go driver's `startReport` writes before the tackler and the arbiter run do not end the hold.
  A park marker does not end the hold while the newest settle record at this stop key later than the stop is of action `hand-over`: that marker is the Go driver's hand-over, whose own hand-back writes no settle record, and the hold runs on to the successor's own end conditions or the backstop.
  A successor that hands back appends a `hand-back` settle record after the hand-over, so its park marker ends the hold.
  Its preconditions are checked on every pass, so switching the arbiter off mid-hold ends it at the next pass; spawns cannot run out mid-hold except by an attempt at this stop key, which keeps it.
- When the hold ends, `stateChangeReady` (`internal/battenshed/notice.go`) measures its report wait from the hold's end instead of from the episode's since, since a hold outlasting `noticeReportWait` would otherwise find the wait already spent at its end and send the notice with `report none yet` before the driver's park marker.
  The hold's end time comes from a record where one carries it: the park marker's write time, the write time of that attempt's `envelope.json` (the attempt record carries no hand-on time), the write time of the attempt's `applied` mark for an applied resume, or the backstop's computed time.
  The hold's end time is never earlier than the episode's since.
  The arbiter switched off leaves no record, so the end is the time batten first observed it, and a batten restart after such an end takes its own start time.
  So a watched awaiting stop with a parent notice waits `noticeReportWait` from the hand-on for the driver's park marker, then `noticeRelayWait` from that marker with the decision-record check, and the driver's own relay after a hand-on is not doubled.
  A stop the hold never covered keeps today's rules.
- Error stops keep the marker hold (`arbiterHolds`) only, since their eligibility depends on a repair escalation batten cannot see.
- The orch stencil's guidance to check the round's circling decision before acting stays as a guard against the orch acting on another signal.

**Bound:** the hold ends at the later of the stop's time plus `arbiter_timeout_min` plus batten's report wait and, when an attempt at this stop key exists, that attempt's deadline plus the report wait.
So a tackler or fork that ran first does not shorten it, and a driver that never spawns the arbiter, or dies, still reaches the parent; because the start comes from the status file and the attempt record, and the end from the park marker, the attempt's `envelope.json` or `applied` mark, or the backstop, a batten restart neither resets the start nor loses the end.

**Rationale:** The board entry's "Arbiter and parent double notice".
In the hub's `.lyx/notices.log`, batten's notices for `logging-cleanup` (2026-10-13T09:26:22Z, stop at 09:22:51Z) and `tender` (2026-10-13T15:32:29Z, stop at 15:28:56Z) went with `report none yet`, about 3.5 minutes after the stop, when `noticeReportWait` had passed and `arbiterHolds` held nothing.
In `logging-cleanup` the arbiter was still running after the notice, since the orch's `continue` landed first and refused the arbiter's own verb; that leaves the pre-marker window or a marker reading not alive.
In `tender` the orch's handoff places the arbiter's `continue` at the notice's minute, which also fits the window after the verb removed the marker.
Batten's trace logged no seam error, and the task worktrees, which held the arbiter attempt records and the driver's logs, are gone, so the evidence does not single out one window; the hold covers all three.

### Added after Discussion (operator, 2026-10-14): Darn review perch can be switched off per run

Decision: A single darn run can skip its Darn-Review perch (zero review rounds) through the board entry's run_settings, not only hub-wide. The per-run off switch is settable in a run's local layer; the hub-wide default stays in darn.yaml. This replaces D3's guard-only, hub-only skip_review: the off switch is per-run overridable like the budget keys.

Rationale: The operator wants small darn tasks run with no review round at all, decided per task; a hub-wide switch changes every darn run. Seen 2026-10-14 on tokencount-fork-output, which could not skip its perch.

### Added after Discussion (operator, 2026-10-14): Every perch, loom and darn, can be set to zero rounds per run

Decision: Every review perch in loom (Discussion-Review, Plan-Review, Webster-Review, Merge-Review) and darn (Darn-Review) takes a bounce budget of 0, meaning the perch runs no review round and the run goes past it, settable per run through the board entry's run_settings as well as hub-wide. Today a per-perch key at 0 means 'take review_max_bounces'; that inherit meaning moves to an absent key, so 0 always means zero rounds. This extends the previous added decision (darn per-run off switch) to all perches and both recipes, with one shape for both.

Rationale: The operator wants darn and loom configured the same way and wants to switch off any perch for a given task, for example a small task that needs no Plan-Review or Merge-Review.

### Added after Discussion (operator, 2026-10-14): Perch budgets carry no guard

Decision: No perch budget or perch off switch, in loom or darn, is a guard key: each is an ordinary key, settable to 0 in any layer, the run's local layer included. darn's review_max_bounces and review_hard_max_bounces lose CategoryGuard, and no new guard key is added for switching a perch off.

Rationale: Zero rounds is a normal setting. The guard only protects against a run's own agent switching off its review by editing the local config; an agent doing that is behaving wrongly, and that is to be fixed in the agent, not prevented by config categories (operator).

### Added after Discussion (parent, 2026-10-14): Zero-round perches: settled design

Decision: (1) skip_review is dropped; darn's review_max_bounces: 0 (category none, settable per run) is darn's off switch; D3, D4's skip_review migration, D4's PrepareRunSettings reorder and the acceptance criteria on skip_review, its migration and the refusal of {"darn":{"review_max_bounces":0}} go; darn's minimum becomes 0. (2) Loom's run-wide review_max_bounces also accepts 0, meaning every perch without its own key runs zero rounds; the criterion refusing {"loom":{"review_max_bounces":0}} goes. (3) Inherit is expressed by a null template value (for example discussion_max_bounces: ~, read as *int); null means inherit, 0 means zero rounds. (4) loom's Migrate hook rewrites an existing <perch>_max_bounces: 0 to null in hub and worktree files; a run_settings 0 always has the new meaning. (5) skip_merge_review is retired; merge_max_bounces: 0 replaces it, and skip_merge_review: true migrates to merge_max_bounces: 0. (6) A perch at 0 routes its writer row's Done straight to the row after the perch, as Merge-Review's skip does today; darn at 0 also skips Describe, as today. (7) The hard-cap keys are unchanged: 0 still means inherit. (8) Bound: a task can switch off any review, Discussion-Review included, through its own run_settings or local layer, with no hub guard; this is accepted.

Rationale: The operator wants one shape for darn and loom, zero rounds settable on every perch per run, and no guard on these keys: zero rounds is a normal setting, and an agent misusing it is fixed in the agent.

## Constraints

- `PATTERN-config-layers`: categories stay declared in each module's `ConfigSpec`, built through D1's package, and `skip_review` is classified in the same change; only `configengine` reads layers.
- `PATTERN-hub-wide-config`: darn stays `HubWide`; its local override admits category-none keys only.
- `PATTERN-config-strictness`: darn keeps strict `configengine.Load`.
- `PATTERN-binary-change-reconcile`: D4's hub-file rewrite runs only through `hubreconcile.Ensure`.
- `PATTERN-leaf-packages`: the new package's closed import set is stdlib and `configengine`.
- `PATTERN-refusal-way-forward`: every new or reworded refusal (darn caps by layer, darn `review_max_bounces` 0 at load and in `run_settings`, loom and darn budgets below their bound at upsert, the `budget` awaiting refusal) names its way forward and lands its `refusal-spec.md` row and a test; the `darn.yaml: review_max_bounces below 0` row becomes below 1.
- `PATTERN-no-denied-recovery`: way-forward commands name only verbs the agents may run.
- `PATTERN-orch-pane-single-writer`: batten's notice keeps going through `orchengine.QueueNotice`.
- `PATTERN-test-speed`, `PATTERN-test-economy`: unit tier for config, routing, eligibility and the notice hold, with injected clocks; extend existing tables where one covers the surface.

## Auto-mode assumptions

- No operator answered: every pick is the writer's, recorded in the support log's question ledger.

## Open risks

- `overview-seats` runs in parallel and edits `internal/loomcli/checklist.go`, `internal/loomcli/doc.go`, `contracts/specs/refusal-spec.md` and `configreg`; expect merge conflicts at Merge-In in whichever lands second.
- A darn soft cap above the hub's hard cap, set in a hub file or a task's `run_settings`, passes `lyx board upsert` (which sees only the run's document) and is refused at load, parking the run until a cap is changed; the refusal names the fix.
- An older `lyx` binary reading a `darn.yaml` migrated to `skip_review` refuses the unknown key during a deploy window.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 4: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/discussion/round-4-review.md; fixer report: _lyx/reviews/discussion/round-4-fixer-report.md.
  - d9-hand-on-end-time-file (class design, severity MEDIUM, open in rounds 4)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- A darn `.local` override and a board `run_settings` of `{"darn": {"review_max_bounces": 2, "review_hard_max_bounces": 3}}` are accepted and the darn run's Darn-Review uses budget 2 and hard cap 3.
- `review_max_bounces` below 1 in any darn layer is refused, naming the layer's way forward.
- `skip_review: true` in the hub `darn.yaml` routes Darn-Write to Merge-In with no Darn-Review and no Describe; `skip_review` in a `.local` override or `run_settings` is refused as a guard key.
- A hub `darn.yaml` holding `review_max_bounces: 0` migrates to `skip_review: true` with the template budget, a second migration changes nothing, and a run behaves as with review off.
- A hub `darn.yaml` holding `review_max_bounces: 0` beside `skip_review: false` migrates to `skip_review: false` with the template budget, and its migration line names both keys.
- A `run_settings` of `{"darn": {"review_max_bounces": 0}}` is refused, naming `review_max_bounces` as the key set and `skip_review` in the hub's `darn.yaml` as the way forward, and never naming `skip_review` as a key the task set.
- A hard cap below the soft cap, the soft cap coming from the local layer, is refused naming both the local and the hub route, for darn as for loom.
- Loom's existing config behaviour and refusals are unchanged after the move to the shared package, except that a loom `run_settings` budget below its bound is now refused at `lyx board upsert`.
- A board `run_settings` of `{"loom": {"review_max_bounces": 0}}` or `{"darn": {"review_hard_max_bounces": 0}}` is refused at `lyx board upsert`, naming the key and its bound.
- The arbiter verb admits an awaiting stop with a pending `hard-cap` or `circling` escalation and refuses one with a pending `budget` escalation, an unreadable escalation record or an unknown cause, with the hand-back way forward.
- The Go driver relays the `parent_notice` and hands back, instead of calling the arbiter, at a `budget` escalation and at an awaiting stop whose escalation record is unreadable.
- The incident shape: an awaiting circling escalation gets no batten notice at the three-minute report wait while its arbiter has not yet written its marker, while its marker's strand reads not alive, or after the verb removed the marker and before the driver acts on the verdict.
- Batten sends no notice while a tackler escalates and the arbiter it leads to runs past the stop's time plus `arbiter_timeout_min` plus the report wait, within that attempt's deadline plus the report wait.
- The hold ends at once on the driver's park marker, a hand-on attempt for the stop key, an applied resume at this stop key or the arbiter switched off; it also ends once the backstop passes; a stop report file the Go driver writes before its tackler or arbiter runs does not end it.
- A batten restart mid-hold keeps the original start; a restart after the hold's end and before the notice measures the report wait from the park marker's time, the write time of the hand-on attempt's `envelope.json`, the write time of an applied attempt's `applied` mark or the backstop's time, and from its own start time after the arbiter switched off.
- An attempt hands on at a watched awaiting stop with a parent notice more than `noticeReportWait` after the stop, with no park marker yet, and batten sends no notice until the driver's park marker plus `noticeRelayWait` passes without a decision record, or `noticeReportWait` after the hand-on passes with no park marker.
- A watched `blocked` eligible stop whose Go driver hands over to a fresh driver gets no batten notice before the successor's arbiter hands on.
- A watched `blocked` eligible stop whose successor driver's tackler hands back, with no arbiter attempt, gets its notice on the successor's park marker, not at the backstop.
- The hold does not start when the arbiter is off, its spawns are spent with no attempt at this stop key, or a resume at this stop key was applied.
- Driver stencil: a twice-failed transient spawns the fork first; an awaiting refusal relays the `parent_notice` as a hand-off; no relay follows a `resume`.

## Notes for the plan writer

- Darn config: `internal/darnengine/config.go`, `template.yaml`; routing `internal/loomrecipe/darnrecipe.go`; wiring `internal/loomcli/darnwiring.go`; hard-cap route `internal/loomcli/circling.go` (`perchHardCap`); registry `internal/configreg/configreg.go` and its tests pinning darn's local keys.
- Loom side: `internal/loomengine/caps.go` and the `Categories` map in `internal/loomengine/config.go`.
- Arbiter: `internal/loomcli/arbitereligibility.go`, `drivewiring.go` (`EscalationPending`), `driverules.go`, `arbiter.go` (`PendingEscalation`'s discarded leading value), `circling.go` (`escalationPending`).
- Budget cause sites: `internal/shedadapters/bouncer.go` (`settleUndecided`, `softCapRule`).
- Batten hold: `internal/battenshed/notice.go` (`stateChangeReady`, `arbiterHolds`), wiring in `internal/battencli/wire.go` (`arbiterInFlightFrom`); the verb writes the marker in `recordStartedArbiter` and removes it in `finish` (`internal/loomcli/arbiter.go`).
