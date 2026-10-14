# Perch budgets and stops: darn budget per run like loom, arbiter only when needed

## Goal

Darn and loom read their review budget and off-switch keys through one shared config shape, so a darn run takes a per-run review budget through `run_settings` exactly as a loom run does, and switching Darn-Review off is its own guard key.
The arbiter spawns only at stops that need an opus judgement (D5–D8).
The run's parent hears about an arbiter-eligible stop only once the arbiter has handed on, or when no arbiter will settle it (D9).

## Scope

In:

- `internal/darnengine` config: categories, a new `skip_review` guard key, value checks, a config migration, template comments, `doc.go`.
- A new shared package for the review budget key family, used by both `internal/loomengine` and `internal/darnengine` config; loom's cap type, resolution and cap route move into it.
- `internal/loomrecipe` darn routing, `internal/loomcli` darn wiring, cap routes and circling, `internal/configreg` (darn's migrate and value-check hooks).
- `internal/loomcli` arbiter eligibility and the Go driver's arbiter rule.
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
`darnengine.LoadConfig` validates the resolved stack, and darn registers a `CheckValues` hook in `configreg` so a `run_settings` value below its bound is refused at `lyx board upsert`, as loom's `ValidateLocalValues` does.
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

**Bound:** the switch skips the only review a darn run has, and as a guard it is settable only in the hub's `darn.yaml`, from the prime; a `.local` override or `run_settings` naming it is refused by the existing local-layer check.

**Rationale:** The board entry: the guard stays on the off switch, not on the budget.
The name follows darn's run-wide `review_*` keys; loom's `skip_merge_review` is its per-perch counterpart.

### D4 — A config migration turns an old off budget into the switch

**Decision:** Darn registers a `Migrate` hook in `configreg`: a document holding `review_max_bounces: 0` is rewritten to `skip_review: true` with `review_max_bounces` at the template default; a document without it is unchanged, and running the hook twice changes nothing more.
The hub file is rewritten by `hubreconcile.Ensure` after the binary change, the one reconcile route `PATTERN-binary-change-reconcile` allows; `darnengine.LoadConfig` does not migrate.
A darn load that still finds `review_max_bounces: 0` is refused below 1, and that refusal names `skip_review` from the prime as the way to switch the review off.
`PrepareRunSettings` applies the same hook, so a task's `run_settings` of `{"darn": {"review_max_bounces": 0}}` is refused; the refusal names `review_max_bounces` as the key set and `skip_review` in the hub's `darn.yaml` as the way forward, with a new refusal-spec row and a test.

**Rationale:** A hub whose `darn.yaml` says 0 today keeps its meaning after the binary change.

### D5 — The arbiter's review triggers narrow to `hard-cap` and `circling`

**Decision:** An `awaiting` stop is arbiter-eligible only when its Bouncer row's pending escalation has cause `hard-cap` or `circling`.
A pending escalation with cause `budget`, from any source (unreadable round inputs, a re-escalated round, a run left awaiting since before `judge-cap-call`), and one whose record is unreadable or carries an unknown cause, is refused like an awaiting stop with no escalation and goes to the parent.
One Go function answers this for the verb's eligibility, the Go driver's `EscalationPending` fact and batten's hold (D9), so the three cannot disagree.
The arbiter stencil drops its `budget` rule.

**Bound:** a `budget` escalation now always costs the parent a turn.

**Rationale:** The board entry's case 1: the hard cap and a returning blocker are the arbiter's only review triggers.
Since `judge-cap-call`, the soft-cap call is the Bouncer rule's in `internal/shedadapters/bouncer.go`, so a `budget` escalation is no longer the soft-cap judgement the arbiter was built for.

### D6 — `blocked` and error eligibility stay as they are, case 3 included

**Decision:** The verb's `blocked` rule is unchanged: every `blocked` stop is eligible except a Merge-In or Publish verify halt whose own failure record names a target, already refused since `30385e1ae`.
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
- Batten gains a pre-marker hold: for an awaiting hard-cap or circling escalation, or a `blocked` stop the verb would admit, with the arbiter on, spawns left and no applied resume at this stop key, it holds its state-change notice from the stop's own time in the status file until the arbiter marker exists.
  Eligibility comes from D5's function, wired by `battencli`; `battenshed` holds no copy.
- Once the marker exists, the existing marker hold (`arbiterHolds`) governs, with its own deadline, so a tackler or fork that ran first does not shorten it.
- The hold ends at once when the driver writes a stop report or park marker in this episode, or an attempt for the stop key hands on.
- Error stops keep the marker hold only, since their eligibility depends on a repair escalation batten cannot see.
- Plan finds the cause behind the notice in both incidents and fixes it, whether or not it is the pre-marker window.
- The orch stencil's guidance to check the round's circling decision before acting stays as a guard against the orch acting on another signal.

**Bound:** the pre-marker hold ends at the stop's time plus `arbiter_timeout_min` plus batten's report wait, so a driver that never spawns the arbiter, or dies, still reaches the parent; because the start comes from the status file, a batten restart does not reset it.

**Rationale:** The board entry's "Arbiter and parent double notice": the orch's `continue` landed first and refused the arbiter's own verb.

## Constraints

- `PATTERN-config-layers`: categories stay declared in each module's `ConfigSpec`, built through D1's package, and `skip_review` is classified in the same change; only `configengine` reads layers.
- `PATTERN-hub-wide-config`: darn stays `HubWide`; its local override admits category-none keys only.
- `PATTERN-config-strictness`: darn keeps strict `configengine.Load`.
- `PATTERN-binary-change-reconcile`: D4's hub-file rewrite runs only through `hubreconcile.Ensure`.
- `PATTERN-leaf-packages`: the new package's closed import set is stdlib and `configengine`.
- `PATTERN-refusal-way-forward`: every new or reworded refusal (darn caps by layer, darn `review_max_bounces` 0 at load and in `run_settings`, the `budget` awaiting refusal) names its way forward and lands its `refusal-spec.md` row and a test; row 356 (darn budget below 0) becomes below 1.
- `PATTERN-no-denied-recovery`: way-forward commands name only verbs the agents may run.
- `PATTERN-orch-pane-single-writer`: batten's notice keeps going through `orchengine.QueueNotice`.
- `PATTERN-test-speed`, `PATTERN-test-economy`: unit tier for config, routing, eligibility and the notice hold, with injected clocks; extend existing tables where one covers the surface.

## Auto-mode assumptions

- No operator answered: every pick is the writer's, recorded in the support log's question ledger.

## Open risks

- `overview-seats` runs in parallel and edits `internal/loomcli/checklist.go`, `internal/loomcli/doc.go`, `contracts/specs/refusal-spec.md` and `configreg`; expect merge conflicts at Merge-In in whichever lands second.
- A darn soft cap above the hub's hard cap, set in a hub file or a task's `run_settings`, passes `lyx board upsert` (which sees only the run's document) and is refused at load, parking the run until a cap is changed; the refusal names the fix.
- An older `lyx` binary reading a `darn.yaml` migrated to `skip_review` refuses the unknown key during a deploy window.

## Acceptance criteria

- A darn `.local` override and a board `run_settings` of `{"darn": {"review_max_bounces": 2, "review_hard_max_bounces": 3}}` are accepted and the darn run's Darn-Review uses budget 2 and hard cap 3.
- `review_max_bounces` below 1 in any darn layer is refused, naming the layer's way forward.
- `skip_review: true` in the hub `darn.yaml` routes Darn-Write to Merge-In with no Darn-Review and no Describe; `skip_review` in a `.local` override or `run_settings` is refused as a guard key.
- A hub `darn.yaml` holding `review_max_bounces: 0` migrates to `skip_review: true` with the template budget, a document already holding `skip_review` migrates without conflict, a second migration changes nothing, and a run behaves as with review off.
- A `run_settings` of `{"darn": {"review_max_bounces": 0}}` is refused, naming `review_max_bounces` and `skip_review` in the hub's `darn.yaml`.
- A hard cap below the soft cap, the soft cap coming from the local layer, is refused naming both the local and the hub route, for darn as for loom.
- Loom's existing config behaviour and refusals are unchanged after the move to the shared package.
- The arbiter verb admits an awaiting stop with a pending `hard-cap` or `circling` escalation and refuses one with a pending `budget` escalation, an unreadable escalation record or an unknown cause, with the hand-back way forward.
- The Go driver relays and hands back at a `budget` escalation instead of calling the arbiter.
- The incident shape: an awaiting circling escalation whose arbiter has not yet written its marker, or is running, gets no batten notice at the three-minute report wait.
- Batten sends no notice while a tackler escalates and the arbiter then runs past the pre-marker window; it sends one at once after a hand-on, a driver stop report, an off or spent arbiter, or an applied resume at this stop key; and once the pre-marker backstop passes with no marker; a batten restart mid-hold keeps the original start.
- Driver stencil: a twice-failed transient spawns the fork first; an awaiting refusal relays the `parent_notice` as a hand-off; no relay follows a `resume`.

## Notes for the plan writer

- Darn config: `internal/darnengine/config.go`, `template.yaml`; routing `internal/loomrecipe/darnrecipe.go`; wiring `internal/loomcli/darnwiring.go`; hard-cap route `internal/loomcli/circling.go` (`perchHardCap`); registry `internal/configreg/configreg.go` and its tests pinning darn's local keys.
- Loom side: `internal/loomengine/caps.go` and the `Categories` map in `internal/loomengine/config.go`.
- Arbiter: `internal/loomcli/arbitereligibility.go`, `drivewiring.go` (`EscalationPending`), `driverules.go`, `arbiter.go` (`PendingEscalation`'s discarded leading value), `circling.go` (`escalationPending`).
- Budget cause sites: `internal/shedadapters/bouncer.go` (`settleUndecided`, `softCapRule`).
- Batten hold: `internal/battenshed/notice.go` (`stateChangeReady`, `arbiterHolds`), wiring in `internal/battencli/wire.go`.
- The orch stencil's run-loop description names darn's `review_max_bounces` 0 as off.
