---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:90
    summary: D6 says the orch reads the halted child's way forward as the halt wait reason shown by lyx batten status, but batten status never shows it while the child is halted
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:91
    summary: D6 and its acceptance row point the awaiting child's retry hint at lyx loom resume, which D7 refuses for an awaiting run
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:176
    summary: D6 names lyx loom start --no-attach for a child with no driver strand in reed state, the acceptance row files that case under otherwise and names lyx loom resume
  - id: F4
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:116
    summary: D7 dead-driver-strand way forward names lyx batten run first, which a live batten run refuses as busy, and the acceptance row names only that verb
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:101
    summary: D7 reads driver liveness only and never says how a live driver strand that reed marked retiring is treated, though lyx loom start refuses to adopt it
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:58
    summary: D3 mixes deadline and timeout for one concept and never names the one package that declares the 60-minute constant, which only the support log places in verifytree
  - id: F7
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md:126
    summary: D8 lists no docs/overview.md update though the new verb joins its loom verb list and its batten paragraph says batten cannot restart a halted child's driver
---

### [MEDIUM] D6's venue for the way forward does not show it

**Location:** `_lyx/discussion/decision-record.md:90`
**Issue:** D6 says "the way forward the orch reads for a halted child is the halt wait reason shown by `lyx batten status`".
A halted child is a budget-exempt Stuck on the Run-Shed row, which routes back to itself.
`internal/shedengine/run.go` persists that bounce as state `running` with an empty error, so `Activity.Wait` is empty.
`battenStatusExtras` in `internal/battencli/arm.go` adds `stuck_reason` only when batten's own state is `blocked`, which an exempt wait never reaches.
The reason text reaches the batten log (`reportStuck`'s Warn) and `<producer>-stuck.md` in batten's scratch directory, and nothing the orch reads by command.
The premise is false, and so is the same claim in `docs/overview.md`'s batten paragraph.
What the orch does read is its role stencil and D7's refusal messages, which D7 and D8 already cover.
**Fix:** reword D6's bullet so the wait reason is the text in batten's log and stuck-reason file, say that `lyx batten status` does not show it and that no status surface is added (the notice channel and runs overview own that), and name the orch stencil and D7's refusals as where the orch learns the verb.
Add the `docs/overview.md` sentence to D8's doc list (F7).

### [MEDIUM] The awaiting hint points at a verb D7 refuses

**Location:** `_lyx/discussion/decision-record.md:91`, `_lyx/discussion/decision-record.md:110`, `_lyx/discussion/decision-record.md:176`
**Issue:** D6 says "the same applies to the awaiting child's retry hint", and the acceptance row says a halted or awaiting child's wait reason names `lyx loom resume`.
D7 refuses an awaiting run and names `lyx loom approve` or `lyx loom reject`.
The awaiting hint in `internal/battenshed/innerrun.go` (resume delivered, driver has not re-stepped) tells the operator to run `lyx loom start` if the driver has ended, and that verb is the one that wakes an awaiting driver.
Rewriting the hint to `lyx loom resume` sends the reader into a refusal.
**Fix:** limit D6's rewrite to the halted (blocked, paused, failed) wait reason, keep the awaiting hints on `lyx loom start` and `lyx loom approve`/`lyx loom reject`, and make the acceptance row say halted only.

### [LOW] No-driver-strand wording differs between D6 and the acceptance row

**Location:** `_lyx/discussion/decision-record.md:176`
**Issue:** D6 says the halt reason names `lyx loom start --no-attach` once a revive has failed or when the child has no driver strand in reed state.
The acceptance row names it only for a failed revive and files everything else, the no-driver-strand child included, under `lyx loom resume` "otherwise".
**Fix:** make the acceptance row state both cases the way D6 does.

### [LOW] The dead-strand way forward can name a verb a live batten run refuses

**Location:** `_lyx/discussion/decision-record.md:116`, `_lyx/discussion/decision-record.md:178`
**Issue:** `lyx batten run <slug>` is refused with "another batten run already holds the run lock" while a batten run is live (`internal/battencli/arm.go`).
A live batten run revives a dead driver strand itself, once per episode, so in exactly the case where `lyx loom resume` sees a dead strand under a live batten, the first named step either happens on its own or has already been spent.
`lyx loom resume` reads no batten state and cannot tell which case it is in.
D7 states the conditions as "a run batten does not hold or once D6's revive has failed", and the acceptance row requires only `lyx batten run <slug>` to be named.
**Fix:** have the refusal name both verbs with their conditions: `lyx batten run <slug>` from the prime when no batten run is watching the child, `lyx loom start --no-attach` in the task worktree when one is watching or its revive failed.
The acceptance row requires both to be named.

### [LOW] Retiring driver strand is not classified

**Location:** `_lyx/discussion/decision-record.md:101`
**Issue:** D7 reads "strand liveness" and shares `lyx start`'s parked branch.
`resolveDriverStrandAction` in `internal/loomcli/bootstrap.go` splits a live strand into live and live-but-retiring, and `lyx loom start` never adopts a retiring one.
A resume verb that reads only liveness would type the resume line into a pane already marked for removal.
**Fix:** state in D7 that a retiring driver strand counts as not live, so it falls under the dead-strand refusal, and add it to that acceptance row.

### [LOW] Verify deadline terms and declaring site

**Location:** `_lyx/discussion/decision-record.md:58`, `_lyx/discussion/decision-record.md:171`
**Issue:** D3 uses "deadline" for the parameter and the expiry and "timeout" for what the findings name, in adjacent sentences, and the acceptance row says "naming the timeout" next to "outlives the deadline".
D3 says "production callers pass a named constant" without saying where it is declared, and the plan writer reads only the decision record.
Q3 of the support log places it in `verifytree`, which the plan writer never sees, so several production callers could each declare their own copy.
D3 and the Constraints already agree that a test passes a short value and the production callers pass the constant.
**Fix:** use "deadline" throughout the record and say that `verifytree` declares the one 60-minute constant the production callers pass.

### [LOW] D8 omits `docs/overview.md`

**Location:** `_lyx/discussion/decision-record.md:126`
**Issue:** `docs/overview.md` lists the loom verbs and says batten "cannot restart the driver of a halted child".
D7 adds a loom verb and D6 makes the second statement false, and CLAUDE.md requires such docs to land in the same commit.
D8 names only the `doc.go` files, the orch stencil and `PATTERN.md`.
**Fix:** add `docs/overview.md` (the loom verb list and the batten halted-child paragraph) to D8.

## Focus verdicts

- D7 lock-held refusal: not a defect.
  Its way forward is `lyx loom resume` again in a few seconds, which the orch can run, and the parenthetical is correct that `lyx loom status` shows no lock holder.
  The Constraints and Acceptance wording agree with D7 and with each other.
- D7 against D6's revive: both cases are covered.
  A dead driver strand present gets `lyx batten run <slug>` or `lyx loom start --no-attach`, and no driver strand gets `lyx loom start --no-attach`, which spawns a driver for a Go-driven run and an LLM one alike.
  The acceptance row has a case for each; the wording gaps are F3 and F4.
- Batten's text for the orch: F1 and F2.
  D6 and D7 agree on `lyx loom resume` for the halted wait reason, and D8 lists the `doc.go` sentence.
- Verify deadline: F6.
  The supplier is stated (production callers pass the constant, a test passes a short value) and the Constraints agree with D3.
- Acceptance rows against D7's order: every refused state has a row, and F4 and F5 are the only gaps in a row's wording.

## Focus departures

- The first focus entry says loom "registers no `StatusExtras` hook".
  `internal/loomcli/arm.go` registers `loomStatusExtras`, which adds `pause_requested`, `slug` and `parent` and nothing about a run lock, so the conclusion the entry points at holds and the verdict above rests on the code as read.
