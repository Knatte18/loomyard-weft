---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D2 (step 2, Bound, Rationale) and the D2 acceptance criterion on finalize evaluations
    summary: A finalize evaluation reached off the writer's idle boundary, such as the deadline finalize D2's own bound counts, spawns a fixer into a worktree the live writer is still editing, which falsifies D2's premise that the writer sits idle.
    fix: In D2, add a rule that the wrapped closure spawns a fixer only at an evaluation reached at the writer's idle turn boundary; at a final evaluation reached any other way (deadline, liveness or any finalize not at an idle boundary) the wrap only waits on a live fixer record and then runs the inner closure, spawning none. State how the wrap knows which kind of evaluation it is in, for example a flag the loop sets before finalize's evaluateGate, since a Gate closure takes no argument. Then correct D2's deadline overrun bound to one fixer timeout plus the inner gate runs that remain, correct the attempts bound to match, and change the D2 acceptance criterion "a finalize evaluation runs the fixer at most once" to say that a finalize evaluation off the idle boundary spawns no fixer. Keep the Expired-probe restart path, where the writer is idle, as it is.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D8 first bullet, with D2's Expired probe and the D2 restart acceptance criterion
    summary: D8 keys the row fixer's record set to the row visit, but no row-visit identity exists, since the run scratch directory is per run, and D8 does not say whether a crash re-step of the same row keeps the set, which the D2 restart path depends on.
    fix: In D8, name the identity that keys a row visit and where it is read, backed by the code it rests on (for example the step number of the shed step record the row runs under, read from the run's steps directory under shedrun.ScratchDir), and state that a crash re-step of the same row (resume re-stepping an interrupted step) keeps the same set, so the Expired probe and the wait-only entries see the live fixer, while a goto, reset or fresh row entry starts an empty set.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D1 second bullet (runproc registration), with D8's goto clause
    summary: D1 claims ending the run's processes by cancel, goto or reset stops a live fixer, but goto ends no processes, so a fixer left live by a crashed driver keeps editing the worktree under the next row while D8 gives the new visit an empty record set.
    fix: In D1, replace "by cancel, goto or reset" with the verbs that call runproc's End, EndSparing or EndRole today, and add a decision for goto and reset, in which either they end the run's fixer role through runproc.EndRole before moving the run, or, before its first evaluation, a new visit's wrapped entries stop any live fixer record the run's earlier visits left. Add an acceptance criterion that a goto past a row with a live fixer leaves no fixer running.
  - id: F4
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D4 Bound (role guard) against D4 Rationale, with D1 and D6
    summary: D4's hand-off guard tests webster's verify-fixer role, so the gate verb learns of webster, contradicting D4's own rationale; no decision names which module owns the set of fixer roles the guard tests.
    fix: In D1, state that internal/gatefixer owns the set of fixer roles, its own gate-fixer constant plus each role a caller names to it, with webster's verify-fixer role declared in or registered with gatefixer. In D4's Bound, say that the front tests the caller's role against that set through agentname.MatchesRole, so gatecli imports nothing of websterengine; and reword D4's rationale to match.
  - id: F5
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D3 second bullet and D4 opening, against Scope Out
    summary: The new gate.yaml fixer field is accepted on every gate key with no stated bound, so a hub setting it on darn-write, drift-verify or code-burler-fix makes those agents' own in-work gate calls hand off, which Scope Out excludes.
    fix: In D3, either refuse the fixer field at config load on every gate key but card, with the key and a way forward, or state the bound, namely that on any key it hands off only capped calls of non-fixer roles, and what it then lets through on keys a gated row also runs. Make Scope Out's darn-writer line agree with the choice.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D1 second bullet against D4 step 1
    summary: D1 registers every fixer with runproc under its run, but D4's card fixer is spawned by a detached runner that belongs to no run, and the record never says which owner the card fixer registers under, or what happens outside a hub, where the runner registers nothing.
    fix: In D4, state that the card fixer registers under the same runproc owner the detached runner registers under (the caller's pair slug through originSlug), and what a card fixer does where there is none, either spawned unregistered or no hand-off at all.
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D4 step 1 with D1 (prompt directives)
    summary: D1 renders the parent directive as the webster cold fixer does, from a parent name, but D4 hands the detached runner only the caller's role, so the card fixer's parent is never set and the record does not say whether the no-parent variant is intended.
    fix: In D4, name the card fixer's parent, for example the fork's own parent written into the run request by the front beside the role, or state that the card fixer renders the no-parent directive and ends dead rather than asking.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D7 first bullet
    summary: D7 accepts the lens-review staleness a fixer causes only for Plan-Write, yet the Discussion writer also runs its lens review right after writing and its fixer then edits the files that review read.
    fix: In D7, make the staleness sentence cover both writers, saying that a plan or discussion fixer may edit files the lens review read, that the staleness is accepted, and that D5's Bound names what judges those edits.
---

### [BLOCKING] A finalize evaluation off the idle boundary spawns a fixer beside a live writer

**Location:** `_lym/discussion/decision-record.md` D2, with its deadline bound and the acceptance criterion "a finalize evaluation runs the fixer at most once".

**Issue:** D2's Rationale reads "the writer sits idle at its turn boundary, so the fixer edits a worktree nobody else is touching".
The wait loop finalizes on the deadline whatever the writer is doing (`internal/shuttleengine/wait.go:416-420`).
`finalize` runs `evaluateGate(true)` before it removes the writer's strand (`wait.go:1781-1784`, removal at `wait.go:1830-1836`).
A must-pass entry cannot set `Final` (`gate.go:56-58`, `wait.go:1625-1626`), so finalize runs the wrapped `Gate` or `Forked` closure.
That closure takes no argument (`gate.go:44`), so it cannot tell a final evaluation from one at the turn boundary.
D2's own overrun bound counts exactly this path: an evaluation fails, the writer is re-prompted with budget left and starts editing, and then the deadline finalize "runs the wrapped closure again".
At that point the wrap spawns a fixer while the writer is mid-turn.
The re-run then judges a tree both agents are changing.
On Plan-Write and Discussion-Write, which have a single entry each (`contracts/recipes/tapestry-recipe.yaml:122-124`, `:268-270`), both agents edit the same files.
D7 makes this path reachable on every failure of those two rows.
All four lenses found this independently, as did the handler.

### [MEDIUM] The row visit that keys D8's record set is undefined

**Location:** `_lym/discussion/decision-record.md` D8 first bullet, with D2's `Expired` probe and the D2 restart acceptance criterion.

**Issue:** The only scratch directory the row's Env carries is per run, with no visit segment: `shedrun.ScratchDir` (`internal/shedrun/paths.go:155-157`), `RunScratchDir` (`internal/shedrecipe/recipe.go:91-93`, `internal/loomcli/wiring.go:627`).
D8's promise that "a goto, reset or row re-entry starts an empty set" therefore rests on an identity the record never names.
D8 is also silent on resume, which re-steps an interrupted step.
If that re-step counts as a new visit, the `Expired` probe and the wait-only entries never see the live fixer.
Darn-Write's `clean` entry then fails over the fixer's uncommitted edits, which is the defect D2's wait-only wrap exists to prevent.
Merged from walkthrough F2, existing-paths F3 and bound-breaker F3.

### [MEDIUM] Goto does not stop a live fixer

**Location:** `_lym/discussion/decision-record.md` D1 second bullet, with D8's goto clause.

**Issue:** D1 says registration with runproc means "ending the run's processes, by cancel, goto or reset, stops it".
The callers of `runproc.End`, `EndSparing` and `EndRole` are tender, pair teardown, batten cancel, fabric, orch, reed and beam paths.
`internal/shedverbs/goto.go` holds no runproc or strand-stop call (handler grep).
Case: the driver dies while a closure blocks on a fixer, and the operator runs goto to another row.
D8 gives the new visit an empty set, so no entry waits on the old fixer, attaches to it or stops it, and it keeps editing the worktree under the next row.
Merged from existing-paths F2 and bound-breaker F2.

### [MEDIUM] D4's role guard teaches the gate verb webster's role

**Location:** `_lym/discussion/decision-record.md` D4 Bound and Rationale, with D1 and D6.

**Issue:** D4's guard names webster's `verify-fixer` role (`internal/websterengine/verifyfixer.go:26`), while D4's Rationale says "the gate verb learns nothing of webster".
`PATTERN-agent-name` makes a spawned role its module's constant, so gatecli would either import websterengine or repeat the literal.
D1 lets a caller "name its own role", but no decision says which module owns the full set the guard tests.
Merged from interactions F2, bound-breaker F4 and the handler's own reading.

### [LOW] The gate key `fixer` field has no stated bound on which keys carry it

**Location:** `_lym/discussion/decision-record.md` D3 and D4, against Scope Out.

**Issue:** D3 lets any gate.yaml key carry `fixer`, and D4's guard keys only on capped calls and fixer roles (`internal/gatecli/detachfront.go:232-264`).
A hub that sets `gates.darn-write.fixer` makes the darn writer's capped in-work calls hand off, which Scope Out excludes.
The attack-surface rule asks the introducing Decision to state what the field lets through and what bounds it.
Merged from bound-breaker F5, interactions F4 and the handler's own reading.
It is kept at LOW: only a hub's own config reaches it, and those agents' calls block the same way the fork's do.

### [LOW] The card fixer's runproc owner is unnamed

**Location:** `_lym/discussion/decision-record.md` D1 second bullet and D4 step 1.

**Issue:** `registerRunner` registers the detached runner under the pair slug, and registers nothing outside a hub or without a slug (`internal/gatecli/detachfront.go:807-818`).
D1's "registered with `internal/runproc` under its run" has no run to point at for a fixer spawned by that runner.

### [LOW] The card fixer's parent directive input is unnamed

**Location:** `_lym/discussion/decision-record.md` D4 step 1 and D1.

**Issue:** The runner's environment drops the strand name (`internal/gatecli/detachfront.go:130`).
D4 writes only the caller's role into the request, so the parent directive D1 renders has no parent name to render, and the record does not say whether that is intended.

### [LOW] D7 accepts lens-review staleness for one writer only

**Location:** `_lym/discussion/decision-record.md` D7 first bullet.

**Issue:** "A plan fixer may then edit files the lens review read … that staleness is accepted" covers Plan-Write alone.
The same bullet has the Discussion writer run its lens review right after writing, and D5 lets its fixer edit the three discussion files.

## Rejected

- Open risk "the 30-minute timeout is unmeasured for the card key" as a deferred research question: the record shows no card call is on record to measure, and states it as an accepted risk that R1 measures, rather than leaving it for Plan or an implementer to find out.
- D5's lens-review-only skip as Go sorting findings by code: the lens gate emits a structured `lens-review-missing` check (`internal/loomshed/gates.go:319-328`), so the skip needs no findings-text parsing and matches D5's single stated exception.
- Leanness and relocation: the Constraints evidence for the 594 s measurement is the research the deferred-research rule asks for, the support log holds the deliberation, and no decision-record passage is narrative a plan writer does not need.
