---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:70
    summary: D4 removes the tamper check for batten without stating what it now lets through or what bounds it; the stop-key check bounds run movement, not tackler writes in the prime, yet a batten resume can re-enter Land.
    fix: In D4, replace the tamper sentence with the removed guard's bound. State that with no snapshot, a batten tackler that writes in the prime (files, index, a local commit) has its verdict acted on, and that a resume may then re-enter Land or Worktree-Teardown over that state. Then either name the mechanical bound that replaces the snapshot (for example a narrowed batten snapshot of the prime's tracked dirty paths outside the records side, taken and compared by the driver, with prime HEAD movement excluded because landings move it), or state plainly that the only bound is the tackler stencil's write rule plus the agent settings' denies, and add that to the D4 Bound line. Add an acceptance criterion for whichever bound is chosen.
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:87
    summary: D5's Migrate rewrite of values equal to the old defaults is stateless and runs on every reconcile and on every task's run_settings, so it permanently erases any deliberate pin of those values, including a run_settings pin that overrides a hub value; D5 states no bound and does not change the loom template.
    fix: In D5, replace the Migrate sentence with a decision that states its scope and bound. Name that loom's Migrate hook runs on every reconcile and on run_settings documents (internal/configreg/runsettings.go calls module.Migrate), so a value-equality rewrite is permanent and also fires on a task's run_settings. Pick one of - (a) rewrite only the hub-level loom.yaml once, keyed on a marker the rewrite leaves (for example only when all four keys equal the old defaults together with the old template comment), never on run_settings; or (b) keep the value rewrite and state the bound - an operator can no longer pin llm, haiku, 10 or 4 at the loom level, and a run_settings pin of one of those values falls through to the hub layer. Also state that the loom template's four keys ship with the inherit value and that darn.yaml's four and batten.yaml's three new keys ship with the inherit value, so a fresh hub does not mask shed.yaml. Extend the Migrate acceptance criterion to the chosen scope, including a run_settings case.
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:85
    summary: D5 gives darn.yaml the three tackler keys, but no Decision says that a darn run driven through lyx loom drive resolves its tackler settings from darn.yaml; D3 mounts the body with loom's seams, D6 covers only driver_kind, and today's drive wiring resolves the tackler from loom's config for every run.
    fix: In D5, after the Resolution sentence, add that the arming module passes the override of the run's own recipe - loom.yaml for a loom run, darn.yaml for a darn run, batten.yaml for batten's own run - for all tackler keys, chosen by the seed's recipe at lyx loom drive's wiring, the same rule D6 states for driver_kind. Add an acceptance criterion - a darn run driven by lyx loom drive with darn.yaml tackler_max_spawns 0 and loom.yaml 4 spawns no tackler, and the reverse case spawns one.
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:71
    summary: D4 restricts a batten tackler resume to the halted row, but today's verdict validation accepts the routed-from row as a goto target, and neither D2's seam list nor the acceptance criteria carry the restriction, so nothing tells the plan where it is enforced or how it is checked.
    fix: Add a seam to D2 - Resume goto targets - loom and darn allow the current row and the routed-from row as today; batten allows the current row only, and a verdict naming any other row is refused and hands back. In D7, state that the stencil's goto rule reads from that seam's allowed rows. Add to the batten acceptance criterion that a batten resume whose goto names the routed-from row is refused and hands back.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:52
    summary: D2's Bound line claims every unwired path ends in a hand-back, but an unwired verify-failure target or hand-over spawn makes its row never fire, so the stop falls through to later rows (the tackler or a re-step), not to a hand-back.
    fix: Rewrite D2's Bound line as - an unwired seam removes a row or an agent from the table; a stop whose row never fires meets the later rows unchanged, so it ends in the tackler under its spawn budget and stop-key check, a re-step under the repair cap, or a hand-back, and no unwired seam adds a resume path the table does not already have.
  - id: F6
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:60
    summary: D3's premise that no go-seeded run exists is false - every batten run's seed records driver go, including this task's own live batten run - so the no-migration conclusion holds only because batten writes none of the driver records today, and D4 does not say what a batten run in flight across the deploy does.
    fix: In D3, replace the premise sentence with - No run has driver records today - loom and darn runs are all seeded llm, and batten runs, though seeded go, write none of the driver records - so the records move to shedrun paths without a migration. In D4, add one sentence for a batten run started on the old build - a lyx batten run of the new build over it takes the driver lock, and a step the old process still runs is adopted through the loop's waiter like any detached step.
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:154
    summary: Behaviour D2 and D7 add has no checkable criterion - batten's hand-over never firing on a changed build, batten's verify-goto row never firing, and the stencil naming no arbiter and carrying the stop list and reading order.
    fix: Add to Acceptance criteria - a batten stop after the on-disk build changed goes to the tackler, never a hand-over; a batten blocked stop with a Publish failure record present goes to the tackler, never a verify goto; the rendered tackler prompt for each recipe names no arbiter and contains the stop-kind list and the reading order D7 names.
  - id: F8
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:83
    summary: D5 states what a set tackler_max_spawns 0 means in a module override but not tackler_timeout_min 0, which today means take 10 minutes; under overrides it could inherit shed.yaml or pin 10.
    fix: In D5, after the tackler_max_spawns 0 clause, add that a set tackler_timeout_min of 0 in a module or in shed.yaml means the built-in 10 minutes, or that it inherits the next layer - pick one - and add the case to the overrides acceptance criterion.
  - id: F9
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:74
    summary: D4's keep-behaviour sentence names abandonedSession twice, once in the status envelope list and once for lyx batten run's envelope.
    fix: In D4, cut the clause and lyx batten run's envelope keeps abandonedSession from the sentence, and name lyx batten run's envelope in the list instead of the bare status envelope if that is the envelope meant.
---

### [BLOCKING] Batten's removed tamper check has no stated bound

**Location:** `_lyx/discussion/decision-record.md:70`, with the D4 Bound line at `:75` and D2's Tamper snapshot row at `:48`.

**Issue:** The rubric's attack-surface rule requires a Decision that removes a guard to state what it now lets through and what bounds it.
D4 gives the reason for dropping the snapshot (other writers in the prime) but not the consequence.
In today's driver (`internal/loomcli/drive.go`, `tackle`), the snapshot comparison is the only mechanical check that a tackler left the worktree as it found it; the tackler stencil's write rule (`contracts/stencils/loom/loom-template-tackler.md`, What you may change) is prompt text.
The stop-key check D4 offers as the bound compares the run's status (`recheckAfterVerdict`), so it catches a moved run, not a tackler's writes.
For batten the worktree is the prime, and D4's own resume line lets a resume re-enter Land and Worktree-Teardown, so a tackler's stray change in the prime meets the landing path unchecked.
The focus question whether the stop-key check alone is a sufficient bound for a batten resume: it is not, since it covers a different hazard.
The arbiter removal (escalate hands back) and the hand-over removal (row never fires, batten keeps its build as today) are bounded.

### [BLOCKING] The loom.yaml Migrate rewrite is permanent and reaches run_settings

**Location:** `_lyx/discussion/decision-record.md:87`.

**Issue:** `configreg.Module.Migrate` rewrites a present file's bytes before every reconcile, and `internal/configreg/runsettings.go` runs it on every task's `run_settings` document too; loom's `MigrateConfig` keeps no record of having run.
A rewrite keyed on value equality therefore fires forever: an operator who sets `tackler: haiku` in `loom.yaml` to pin it against a later `shed.yaml` change sees it erased at the next reconcile.
In `run_settings` it inverts intent: with a hub `loom.yaml` of `tackler_max_spawns: 0`, a task that sets `tackler_max_spawns: 4` to turn its tackler on gets the value rewritten to inherit and falls through to the hub's 0.
D5 states none of this, so the decision is unbounded under the rubric and wrong as stated for the run_settings path.
The loom template (`internal/loomengine/template.yaml`) also spells the four defaults, and D5 does not say the template switches to the inherit value, so a freshly seeded hub reproduces the masking D5's rationale exists to remove.

### [MEDIUM] No Decision names where a darn run's tackler settings come from

**Location:** `_lyx/discussion/decision-record.md:85`, read against D3 at `:56` and D6 at `:93`.

**Issue:** Today `goDriverDepsFor` in `internal/loomcli/drivewiring.go` calls `loomengine.ResolveTackler(c.cfg, ...)` on loom's config for every run `lyx loom drive` serves, darn included.
D5 adds the tackler keys to `darn.yaml` and D6 moves `driver_kind` to per-recipe resolution, but only `driver_kind`.
D3 mounts the body "with loom's seams" for darn runs, which reads as loom's config.
On the focus question whether D3, D5 and D6 agree on which module's override each recipe reads: they agree for `driver_kind` and for batten (D5's `batten.yaml` has no `driver_kind`, D6's child-driver reader picks by the child's recipe, consistent), but leave the darn tackler keys open, so `darn.yaml`'s new keys may be dead.

### [MEDIUM] The halted-row-only resume for batten has no home

**Location:** `_lyx/discussion/decision-record.md:71`.

**Issue:** `internal/loomcli/tackler.go` validates a verdict's goto as the current row or the routed-from row, for every run.
D4 narrows that for batten, which is recipe-specific behaviour, yet D2's seam list, which D1 makes the only route for recipe-specific behaviour, carries no seam for it, and the batten acceptance line checks only that a resume re-enters the halted row, not that a routed-from goto is refused.
The D4 Bound line leans on this restriction, so leaving its enforcement unplaced weakens the bound too.

### [LOW] D2's Bound line overstates where unwired paths end

**Location:** `_lyx/discussion/decision-record.md:52`.

**Issue:** In `decideDrive` (`internal/loomcli/driverules.go`), an unwired verify-failure target leaves `VerifyMatches` false and an unwired hand-over leaves its row unmatched, so the stop continues to the resume-returned, transient, interrupt and tackler rows.
D2's own row texts say "that row never fires", which contradicts "every unwired path ends in one".
The focus question on the watched read and the friction writer: both hold — batten never watched only drops the watched hand-back variant, which needs a verify match batten never has, and a paused stop hands back with or without the note.
The claim's intent (no unwired seam adds a skip of a hand-back) holds; its stated reason does not.

### [LOW] D3's no-go-seeded-run premise is false for batten

**Location:** `_lyx/discussion/decision-record.md:60`.

**Issue:** Every batten seed under `loomyard-weft/_lyx/shed/` records `"driver": "go"`, this task's own included, while the child seeds in the live pairs record `llm`.
The no-migration conclusion survives only because batten writes no driver lock, park marker, settle records or tackler attempts today, which the record does not say.
On the focus question whether the driver-lock refusal and the adoption of a detached step agree: they do — a dead process releases the driver lock, and the waiter in `internal/shedverbs` adopts a live loop by its pid file — but D4 does not cover a batten run already in flight when this lands.

### [LOW] Acceptance criteria miss behaviour D2 and D7 add

**Location:** `_lyx/discussion/decision-record.md:154`.

**Issue:** On the focus question of matching criteria: D8 and D1's import bar have criteria, D7's recipe marker has one, and no criterion asserts something no Decision states.
Missing are batten's never-firing hand-over and verify-goto rows from D2, and D7's stencil changes beyond the marker: no arbiter named, the stop-kind list, the reading order.

### [LOW] The meaning of a set tackler_timeout_min 0 is open

**Location:** `_lyx/discussion/decision-record.md:83`.

**Issue:** `internal/loomengine/config.go` documents a zero `tackler_timeout_min` as taking 10 minutes.
D5 defers only the representation of unset versus 0 to the plan, but which meaning a set 0 carries under the override scheme is a choice, not a representation.

### [NIT] abandonedSession is named twice in D4

**Location:** `_lyx/discussion/decision-record.md:74`.

**Issue:** The keep-behaviour sentence lists the status envelope and `abandonedSession`, then repeats that `lyx batten run`'s envelope keeps `abandonedSession`.

## Rejected

- Leanness relocation in D4 and D5 (focus item 7): each sentence of both bodies carries a requirement, a bound or a one-line rationale the plan writer needs; the only redundancy is F9. No sentence is deliberation narrative that belongs in `support-log.md`.
- The arbiter and hand-over removals for batten as unbounded guards: D2 bounds the arbiter (stops it would take hand back, an escalate hands back with its case brief), and the hand-over's absence keeps batten's build for its life as today.
