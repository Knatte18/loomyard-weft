---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:94
    summary: D6 checks only Master's engine before spawn, but the recovery strand's sessions are fork-audited too, so a recovery key naming an engine without fork audit spawns unchecked and the Bound omits this second relaxed guard
    fix: In D6, make the wiring check both Master's and the recovery strand's ModelChoice (the two roles webster resolves) for fork audit before Master spawns, each refusal naming its own key (`master` or `recovery` in webster.yaml); rewrite the Bound to state that both role engines are checked before Master spawns and that the default engine is the only one no longer checked; add an acceptance criterion that a recovery choice on an engine without fork audit is refused before Master spawns.
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:95
    summary: The way forward for D6's lookup-point refusal (set master, then re-run the verb) cannot clear it, because D6 itself says a key change never changes the engine an existing run is read through
    fix: In D6, split the two check points' ways forward. Keep the `master` (and, per F1, `recovery`) way forward for the wiring check only. For the lookup of a recorded engine, state its cause and own way forward - an absent record value reads as the current default, so the way forward is to set `engine:` in shuttle.yaml back to an engine with fork audit, then re-run the verb or re-step the Webster row - and state that once both role engines are checked before spawn, a recorded engine without fork audit is otherwise unreachable and is an internal error, not a refusal row. Add an acceptance criterion for the absent-record case.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:114
    summary: D8 assumes every caller reaching the Requires refusal resolved a config model key, but lyx burler run with a cluster fan reaches it through flag models with an empty engine, so its refusal has no key to name and the way forward does not apply
    fix: In D8, name the `lyx burler run` caller, whose refusal names the default engine and the way forward sets `engine:` in shuttle.yaml to an engine that meets the requirement, then re-runs the verb; add its refusal-spec row in the burler section, and add an acceptance criterion that `lyx burler run` with a cluster-fan profile on a default engine without `forks` is refused before any pane exists with that way forward.
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:81
    summary: D5 marks the loom driver peer-messaging for messages its notify stencils send to the parent its parent directive names, which Scope's Out rule excludes as parent asks, while Discussion-Write's delivery prompt messages its parent the same way and is not marked
    fix: In D5, drop `peer-messaging` from the Claude-strand loom driver row and delete the parenthetical about the driver's notify stencils from the rule, so parent messages never imply peer-messaging as Out states; in Open risks, reduce the `creel-agents` bullet to the seat rows only.
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:45
    summary: D2's scan pins only reads of modelspec.Resolved.Params, so a hand mapping that copies resolved.Model and drops resolved.Engine, the defect D2 exists to stop, passes the scan, and D2 states no such bound
    fix: In D2, widen the rule to no production read of any `modelspec.Resolved` field (Engine, Model or Params) outside `internal/modelspec` and `shuttleengine.NewModelChoice`; carry the same wording into `PATTERN-model-choice-mapping` and the first acceptance criterion, so the scan fails on a new read of any of the three fields.
  - id: F6
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:149
    summary: The usage-record acceptance criterion omits the engine key only for a pass with no run record, while D4 omits it for a half or a pass
    fix: In the acceptance criterion, change "omits the key for a pass with no run record" to "omits the key for a half or pass with no run record".
  - id: F7
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:134
    summary: The Auto-mode assumptions line points the plan writer at the support log's ledger, which the plan writer never reads
    fix: Delete "; see the support log's ledger" from that bullet.
---

### [BLOCKING] Recovery strand's engine escapes the fork-audit check

**Location:** `_lyx/discussion/decision-record.md:94` (D6)
**Issue:** `loadRunWrites` (`internal/websterengine/runwrites.go:45`) calls `AuditForks` on every recorded recovery session, not only Master's, and `reset` and `recover-batch` read it.
Today the single default-engine check covers both roles, because every run uses the default engine.
Under D6 each role carries its own engine, and only Master's is checked before spawn.
A `recovery` key naming an engine without fork audit now spawns a recovery strand.
The run is refused only later, at the lookup, mid-recover or at `reset`, after the strand has written to the worktree.
D6 reads the recovery strand only for "turn ends" and its Bound says the default-engine guard is "the only guard relaxed", so it leaves this relaxed guard unstated.

### [BLOCKING] Lookup-point refusal's way forward cannot clear it

**Location:** `_lyx/discussion/decision-record.md:95` (D6)
**Issue:** D6 has two check points and gives one way forward for both: set `master` in `webster.yaml`, then re-run the verb or re-step the row.
D6 also decides that "a config change to the `master` key … does not change the engine an existing run is read through".
At the lookup point, the refusal is on a recorded engine, so re-running after that change reads the same record and refuses again.
The refusal can then only be reached from a recovery record (F1) or from a record with no engine, which reads as the current `shuttle.yaml` default.
Neither is fixed by the `master` key, which breaks `PATTERN-refusal-way-forward`.

### [MEDIUM] `lyx burler run` reaches the `Requires` refusal with no model key

**Location:** `_lyx/discussion/decision-record.md:114` (D8)
**Issue:** The `burler run` profile accepts `cluster-fan` (`internal/burlercli/run.go:36`), which makes the reviewer a D5 cluster-fan row (`forks`).
D3 has its model come from `--review-model` with `Engine` empty, so `Start` checks the default engine.
D8 makes "the caller that resolved the role's model key" wrap the refusal and send the operator to `lyx config <module> --set` on that key.
This caller resolved no key, so D8 neither bounds its refusal nor gives it a way forward.

### [MEDIUM] Driver's `peer-messaging` contradicts D5's own rule and Out

**Location:** `_lyx/discussion/decision-record.md:81` (D5, driver row at line 79)
**Issue:** `shed-template-driver-notify.md` sends its message to "the parent session that the directive above names".
`driverPrompt` fills that directive with `parentdirective.Directive(stencilsDir, parentName, false)` (`internal/loomcli/driverprompt.go:51`), so the target is the driver's parent.
Scope's Out says "a role is not marked `peer-messaging` for its parent asks", and D5's rule excludes the parent.
Discussion-Write's delivery prompt (`loom-template-parent-review-delivery.md`) also sends a `SendMessage`, to `parent.Name` (`internal/loomcli/wiring.go:416`), and D5 does not mark it.
The two cases are the same relationship, and D5 classifies them differently.

### [MEDIUM] D2's scan misses an engine-dropping hand mapping

**Location:** `_lyx/discussion/decision-record.md:45` (D2)
**Issue:** D2's rationale is that "the hand mappings are where the engine was dropped".
The scan only catches reads of `Params`.
A future site that writes `shuttleengine.ModelChoice{Model: resolved.Model}`, or sets `Model` alone on a spec, reads no `Params`, drops `Engine` and passes the scan.
Today's sites read `resolved.Model` beside `Params` (for example `internal/loomengine/rework.go:64`, `internal/frictionengine/spec.go:82`), so the omission is a short edit away.
D2 states no bound for this bypass.

### [NIT] Usage-record criterion narrower than D4

**Location:** `_lyx/discussion/decision-record.md:149`
**Issue:** D4 (line 64) says "A half or pass with no run record … omits the key", but the criterion covers only a pass.

### [NIT] Dead pointer to the support log

**Location:** `_lyx/discussion/decision-record.md:134`
**Issue:** The discussion spec makes the decision record the plan writer's only input, so a pointer to the support log's ledger leads the plan writer nowhere.
The list of auto-picks stays.

## Focus notes

- D6: the guard leaves two paths: F1 (recovery strand) and F2 (the way forward at the lookup point).
  Master's own audit and parse are covered by both check points.
- D5 and D8: the Webster Master, twill writer, cluster-fan reviewer and seat rows match their stencils.
  Master's stencil backgrounds its forks and `recover-batch`, the twill directive allows a turn end with forks outstanding, and the chair messages advisors.
  The driver row does not (F4).
  D8's refusal fires at `Start` before any pane exists, so a wrong key skips nothing.
  The empty-key case is checked against the default engine as the criteria state, and setting the named key clears it for every caller that has a key.
  The keyless caller is F3.
- D2: the bypass is F5.
- D3 and D4: the empty-engine burler flags and the omitted engine key agree with the acceptance criteria, apart from F6.
- Whole record: Scope, Decisions, Constraints and Acceptance criteria otherwise agree.
  The only material that belongs elsewhere is the pointer in F7, and the completeness-before-leanness test finds no other deliberation narrative to move.
