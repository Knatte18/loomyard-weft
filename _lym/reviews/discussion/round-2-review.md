---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D7, D8 (Bound), Goal
    summary: Nothing guards the baseline capture under .lym/drift, so the writer or judge can edit it and containment by code passes at both the drift entry and Drift-Verify
    fix: In D7, add that Drift-Await records a digest of the counted baseline directory tree in scope.json when it counts the capture, and that every diff (Drift-Verify, Drift-Recheck, the drift gate entry, lym drift check) first recomputes it and on a mismatch stops (at a row a Stuck, at the entry and check a reported failure) naming the way forward (lym drift capture is not possible past the branch point, so name lym drift defer for the parent, or a recapture verb if one is designed); state the bound in D8 accordingly, and add a D7 acceptance criterion that an edited baseline file stops Drift-Verify naming that way forward.
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D6 (re-entry), D14
    summary: D6 tells an ended expect session by AgentSessionOpen, whose only record runs.json D14 removes; only Drift-Verify writes that one per-run mark today
    fix: In D6, replace "recorded as today's drift producers record an open agent session (AgentSessionOpen)" with a named per-seat session mark kept in a record D14 does not remove (for example an expect-session and a judge-session field in scope.json, each opened at spawn and closed when the session returns), and state that the judge's spawn uses its own mark; in D14 keep runs.json removed but name where the mark moved.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D3, D15 (diff and key_patterns)
    summary: diff and key_patterns are read from hub config at every row, so a hub edit mid-run changes identities or the mode under an already sealed expectation and stored tuples
    fix: In D3, have Drift-Baseline record drift.diff and drift.key_patterns in scope.json beside the scope, and state that every later row, the drift gate entry and lym drift check read the recorded values, never config; add the matching clause to the D3 acceptance criterion (a later config change of diff or key_patterns changes neither the identities nor the mode of the run).
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D2, D5, D11
    summary: Under command mode Drift-Expect still hands a D1 identity listing and D5 and D11 validate entries against D1 identities, which a capture that cannot meet D1's contract does not produce
    fix: In D2, state what command mode uses instead, namely that Drift-Expect hands the agent the baseline case list only, expected.yaml under command mode holds per-case entries and added/removed cases each with a why, and D5's and D11's identity and range validation is skipped there (case names checked only when the capture writes one file per case); add a D2 acceptance-criterion clause for it.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D2 (Bound), D4
    summary: In command mode a judge's does not match stores nothing, so a plain lym loom resume at the same head reruns the compare and a fresh judge until one rules matches; the bound does not state or limit this
    fix: In D2, store each command-mode does not match or unexpected ruling in judged.json keyed by (warp head, case) and make a resumed row at that head with no amendment since a Stuck without a new judge, as D10 does for lines mode; extend the D2 acceptance criterion with that case.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D9 (judge tree guard)
    summary: The judge snapshot covers neither ignored paths (the .lym baseline and scratch captures) nor weft files outside _lym/drift such as the decision record the judge is handed
    fix: In D9, extend the snapshot to the weft checkout's whole tracked tree and untracked paths (not only _lym/drift/) and to the baseline directory's digest from F1's fix, and state that a change there made by the session is a Stuck; update the D9 acceptance criterion to name both.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D9 (judgement.yaml, interrupted judge)
    summary: A judgement.yaml left by an earlier spawn, or half-written by a dead judge, can satisfy the next spawn's form-only judgement entry, and D9 gives no probe/resume/respawn rule for an interrupted judge
    fix: In D9, state that Drift-Verify removes judgement.yaml before each spawn and gives cell ids unique to the spawn (for example prefixed by a spawn id the entry checks), and that an interrupted judge session is probed, resumed when live and respawned otherwise, as D6 does for the expect agent; add both cases to the D9 acceptance criterion.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D9 (guard Stuck)
    summary: The judge-guard Stuck names no way forward, and on resume the next snapshot counts the judge's change as present before the spawn, so the edit survives and can land unreviewed after Drift-Recheck
    fix: In D9, record the pre-spawn snapshot in scope.json and on a guard Stuck name the changed paths and the way forward (the parent reverts them, for example with git checkout <head> -- <path> on the warp, then lym loom resume); state that a resumed row compares against the recorded snapshot, not a new one, until it matches; add the case to the D9 acceptance criterion.
  - id: F9
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md D9, D10, D7
    summary: Several Stucks the record defines name no way forward - a judge does not match (D9), a stored does not match reused (D10), an exhausted judgement entry, and a failing or timed-out current capture at Drift-Verify
    fix: In D9 and D10, state that a does not match Stuck names the same two ways forward D7 names (the parent's lym drift amend, which drops the covering rulings, and the code-fixing route); in D9, state that an exhausted judgement entry is a Stuck whose resume respawns the judge; in D7, state that a failed or timed-out current capture is a Stuck naming its log and lym loom resume; add each to the matching acceptance criterion.
  - id: F10
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D7 (capture lock and slot), rationale
    summary: The existing capture lock refuses after a bounded retry rather than serialising, and a waiting version would deadlock against a held gate slot because D7 names no order between lock and slot
    fix: In D7, state the order (take the gate slot, or inherit the held one, before the capture lock) and that a check finding the lock held waits for it rather than refusing; correct the rationale to match; add a D7 acceptance-criterion clause that a check inside a gate and a fork's check with one free slot both finish.
  - id: F11
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D4, D14, Open risks
    summary: A run with a live old Drift-Baseline or Drift-Verify agent when this lands keeps that pane running the old stencil while the new code-only row runs, and the record says nothing about ending it
    fix: In D4, state that Drift-Baseline and Drift-Verify, on entry, end any chair of the removed agents still registered for the run through runproc's EndRole before doing their own work, and add the case to the in-flight line under Open risks and to the D4 acceptance criterion.
  - id: F12
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D12, D13
    summary: Removing waive leaves an in-flight run with an uncounted capture, or one waived under the old Drift, with no stated outcome or way forward
    fix: In D12, state that a scope record with WaivedAt set is Done at Drift-Verify and Drift-Recheck with the verdict rendering the waiver (D13), and admit lym drift defer, for the parent and operator, at a Stuck Drift-Verify whose capture never counted; extend the D12 acceptance criterion with both.
  - id: F13
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D3, D13, D14
    summary: D3 and D13 read an old partial record's skip list, but D14 removes partial and skip and keeps read-only only the compare, acceptance and waiver fields
    fix: In D14, add the scope record's scope value partial and its skip field to the fields the scope.json model keeps read-only for an old run (D3, D13).
  - id: F14
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D15 (reinterpretation of partial)
    summary: The hub-wide walk reconciles, pruning retired keys, before it applies reinterpretations, so the flag for a partial base cannot name its skip list
    fix: In D15, drop "with its skip list" from the partial flag, or state that the reinterpretation reads the skip list from the board file's committed content before the reconcile's prune (and name where), so the notice still lists the skipped cases.
  - id: F15
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D6 (Bound)
    summary: On a run past Drift-Expect with no seal, a writer can hand-make a weft commit with Drift-Seal and a Warp-SHA trailer equal to the baseline head after its own warp commits, and it passes as the oldest seal
    fix: In D6, state that on a run that passed Drift-Expect with no seal commit (recorded in scope.json when this lands or when Drift-Expect is Done without sealing), any later seal commit is ignored and the expectation stays empty; extend the bound and the D6 acceptance criterion.
  - id: F16
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D6 (sealing)
    summary: Drift-Expect seals even when the warp head has moved off the baseline head, so the fault surfaces only at Drift-Verify after the writer ran
    fix: In D6, state that Drift-Expect refuses to seal when the warp head differs from the baseline head, as a Stuck naming the moved commits and lym drift defer or a reset through a guarded verb as the way forward; add it to the D6 acceptance criterion.
  - id: F17
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D4 and its acceptance criterion
    summary: D4 says a compare row is Done at once only at the last passing head, yet D12 makes it Done under a deferral and no decision says what it does under scope off
    fix: In D4, state that Drift-Verify and Drift-Recheck are also Done at once when the recorded scope is off or the capture was deferred, and add the scope-off case to the D4 acceptance criterion.
  - id: F18
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D8, D12
    summary: D12 admits defer at a Stuck writer with a missing baseline, but D8 never says what the drift entry or lym drift check does when the counted baseline directory is missing
    fix: In D8, state that the entry fails and lym drift check reports when the counted baseline directory is missing, naming the parent's lym drift defer (D12); add it to the D8 acceptance criterion.
  - id: F19
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D14 (drift-verify gate key)
    summary: D14 claims lym config reconcile prunes a hub gate.yaml entry for drift-verify, but gates is a per-name map whose names the operator owns, so reconcile is not shown to prune it
    fix: In D14, either declare drift-verify a retired key of the gate module so the load drops it and reconcile removes it, or say the entry is left in place as one nothing reads; drop the unproven pruning claim.
  - id: F20
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D8 (Rationale)
    summary: The rationale's example cites key dP moving, but D1's default patterns key that cs line as Re, so code never reports a key dP
    fix: In D8's rationale, change "key `dP` of case X moved" to "key `Re` of case X changed from `127069, dP = 7925.267 Pa` to its new value".
  - id: F21
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md D11
    summary: D11 says the verb does one of two things and then lists three
    fix: In D11, change "It does one of two things:" to "It does one of three things:".
---

### [BLOCKING] The baseline capture is unguarded, so containment by code can be bypassed

**Location:** `_lym/discussion/decision-record.md` D7, D8's Bound, Goal.
**Issue:** The baseline lives under `.lym/drift/baseline` (`internal/driftengine/paths.go:62`), outside git, and `.lym` is a symlink outside the tree.
D8 hands the writer each unexpected cell's case, identity, old and new value.
A writer that edits the baseline file so the old value matches the new one passes its own `drift` entry, and Drift-Verify diffs against the same edited file and passes too.
D8's bound says Drift-Verify "repeats the check as the guarantee", but it repeats the check against the same baseline, so the guarantee does not hold.
D7 promises only that the check itself never writes the baseline.

### [MEDIUM] D6's expect re-entry rests on a mark D14 removes

**Location:** D6, D14.
**Issue:** `AgentSessionOpen` reads `runsRecord.SessionOpen` in `runs.json` (`internal/driftengine/records.go:144-151`, `:277-281`).
D14 removes `runs.json`.
Only Drift-Verify writes the mark (`verify.go:565`), so "today's drift producers" is wrong.
The mark is per run, so the expect agent and the judge would share it.
All four lenses raised this.

### [MEDIUM] `diff` and `key_patterns` are not fixed per run

**Location:** D3, D15.
**Issue:** D3 freezes the scope "so a run's scope does not change mid-run", but `diff` and `key_patterns` are read from hub config.
Editing a pattern between Drift-Expect and Drift-Verify turns `Re` into `#n`, so the sealed `keys` entries cover nothing and stored tuples stop reproducing.
Flipping `diff` moves a running run between containment by code and the opaque judge.

### [MEDIUM] Command mode validates against identities it lacks

**Location:** D2, D5, D11.
**Issue:** `command` mode exists for a capture that "cannot meet D1's contract".
D5 still lists D1 identities and refuses unknown cases and keys, and D11 validates amendments by D5's rules.
So under that mode, the listing and the refusals rest on a reading the capture does not support.
This is the focus directive's command-mode question for expectations and amendments.

### [MEDIUM] Command mode lets a resume re-roll a regression ruling

**Location:** D2's Bound, D4.
**Issue:** No command-mode ruling is stored, and D4 makes a resumed Stuck at an unchanged head check again.
So `lym loom resume` after `does not match` spawns a fresh judge with no code change, until one rules `matches`.
D2's bound names the loss of stored rulings but nothing that limits this retry.

### [MEDIUM] The judge snapshot misses ignored paths and most of the weft

**Location:** D9.
**Issue:** The snapshot covers the warp's tracked tree and untracked paths, plus the weft's HEAD and `_lym/drift/`.
It does not cover the baseline under `.lym`, which git status never lists, or uncommitted weft edits outside `_lym/drift/`, such as the decision record the judge reads as evidence.

### [MEDIUM] A stale or half-written `judgement.yaml` can pass the next judge's entry

**Location:** D9.
**Issue:** `judgement.yaml` stays under `_lym/drift/`, and the guard exempts it.
The `judgement` entry checks form only.
A Drift-Recheck judge whose cell ids match the earlier spawn's can therefore pass on the old file, and that stores rulings no judge made.
D9 also gives no rule for an interrupted judge, though D6 gives one for the expect agent.

### [MEDIUM] The judge-guard Stuck names no way forward, and the edit survives a resume

**Location:** D9.
**Issue:** The Stuck does not tell the parent what to revert.
On resume, the next spawn's snapshot treats the judge's change as "present before the spawn", which D9 says is not a Stuck.
At Drift-Recheck, after the perch, such an edit can land unreviewed.

### [MEDIUM] Several Stucks name no way forward

**Location:** D9, D10, D7.
**Issue:** The focus directive asks for each Stuck's way forward.
These Stucks name none:
- the judge's `does not match` or `unexpected` (D9);
- a reproduced stored `does not match` (D10);
- an exhausted `judgement` entry (D9 is silent, while D5 states it for `expectation`);
- a failing or timed-out current capture at Drift-Verify (D7 is silent).

The amend route already exists, because D11 drops the covered rulings, but the record does not name it at these Stucks.

### [MEDIUM] The capture lock refuses instead of serialising, and no lock order is set

**Location:** D7 and its rationale.
**Issue:** `RunCapture` takes the lock with `AcquireWriteLockWithin` and refuses through `lockHeldRefusal` (`internal/driftengine/capture.go:55-76`), so a second check fails instead of waiting.
A waiting version would deadlock: one check holds a slot and waits for the lock, while the other holds the lock and waits for a slot.
D7 names no acquisition order.
Merges walkthrough F2 and bound-breaker F9.

### [MEDIUM] Old agents live at landing are left running

**Location:** D4, D14, Open risks.
**Issue:** Today Drift-Verify probes and resumes a live verify agent (`verify.go:125-138`), and Drift-Baseline's agent records the scope.
Under D4 both rows are code-only.
A run parked with a live old agent keeps a pane running the old stencil, which edits code and calls removed verbs, and the record never says to end it.

### [MEDIUM] Removing `waive` strands uncounted and already-waived in-flight runs

**Location:** D12, D13.
**Issue:** Today a run whose capture never counted halts naming `lym drift waive` (`verify.go:171`), and the darn recipe comment settles such a run the same way (`contracts/recipes/darn-recipe.yaml:53-54`).
D12 admits `defer` only when a counted capture's directory is missing.
A run with a `WaivedAt` recorded under the old Drift has no stated outcome at Drift-Verify.

### [MEDIUM] The in-flight `partial` rule needs a field D14 removes

**Location:** D3, D13, D14.
**Issue:** D3 leaves out "the cases its recorded skip list names", and D13 renders them.
D14 removes `partial` and `skip` and keeps read-only only "the old compare, acceptance and waiver fields".
`ScopeRecord.Skip` is in neither list (`records.go:73-74`).

### [LOW] The `partial` flag cannot see its `skip` list

**Location:** D15.
**Issue:** `reconcileHubWide` runs `configsync.ReconcileHubWideAt` (`internal/hubreconcile/walk.go:217`) before `applyReinterpretations` (`:239`).
Once `skip` is retired, the reconcile has already pruned it when the flag runs.

### [LOW] A run with no seal can be sealed later by its writer

**Location:** D6's Bound.
**Issue:** The `Warp-SHA` trailer is message text.
On a run past Drift-Expect with no seal, a writer's weft commit carrying both trailers becomes the oldest seal after the writer's own warp commits.
This applies only to runs in flight when this lands, hence LOW.

### [LOW] Drift-Expect seals over a moved warp

**Location:** D6.
**Issue:** If the expect agent committed on the warp, the seal's `Warp-SHA` differs from the baseline head.
The fault shows only at Drift-Verify, where the way forward, `--discard`, empties the expectation.

### [LOW] D4's "only" rule contradicts deferral and scope `off`

**Location:** D4 and its acceptance criterion.
**Issue:** D12 makes Drift-Verify and Drift-Recheck Done under a deferral.
No decision states what those rows do under scope `off`.

### [LOW] D8 is silent on a missing baseline

**Location:** D8, D12.
**Issue:** D12 admits `defer` at a Stuck writer whose baseline is missing, but D8 never says how the entry reacts to one.

### [LOW] The `drift-verify` gate key prune is unproven

**Location:** D14.
**Issue:** `gates` is a per-name map whose names the operator owns.
The walkthrough lens found that reconcile's prune is passed the per-name maps, so an operator's entry there is kept.

### [LOW] D8's rationale names a key D1 never produces

**Location:** D8's Rationale.
**Issue:** D1 itself shows that the `cs` line keys as `Re`, with `dP` inside the value.

### [NIT] D11 counts two actions and lists three

**Location:** D11.
**Issue:** "It does one of two things:" is followed by three bullets.

## Rejected

- Bound-breaker F10, that D1's rationale on repeated keys is weakened: every shifted cell is either covered, and so judged against its `why`, or uncovered, and so unexpected and a Stuck. That is exactly the claim that it costs judge work and lets no regression through.
- Bound-breaker F6, raised as MEDIUM, is kept as LOW (F15): it reaches only runs in flight with no seal, since in a new run the genuine seal is always the oldest.
