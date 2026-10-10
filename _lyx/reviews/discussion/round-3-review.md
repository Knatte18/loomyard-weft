---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:142
    summary: The record counts only the two darn.yaml refusals; the other loom refusals it adds or reroutes (spent Darn verify budget, D9 commit failure, Publish verify-failed way forward, gate-slot sites) get no refusal-spec row or acceptance criterion
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:88
    summary: The Darn stencil gets no input for re-entering after a Publish verify failure or a resumed spent budget, and Publish's existing way-forward clause points a darn run at a Webster-Review round that does not exist
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:41
    summary: D1 says reject only swaps its routing, but reject's state check, refusal text and help name PR-Rework and the plan archive, which a darn run has neither of, and D12 leaves reject's help unchanged
  - id: F4
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:91
    summary: D4 says a lowered verify_attempts halts the next goto and D7 says every arm, but D1 says goto reads routing only and status and pause build nothing
---

### [BLOCKING] Refusals the record introduces or reroutes, beyond the two config refusals, have no spec row

**Location:** `_lyx/discussion/decision-record.md:142` (D7), together with D3 line 75, D4 lines 90-93, D6 line 124, D9 line 165 and the Docs acceptance criterion at line 302.

**Issue:** `PATTERN-refusal-way-forward` requires every refusal that loom's verbs can reach to name its way forward in its message and to have a row in loom's section of `contracts/specs/refusal-spec.md`. Each new one must also land with a test that reaches it.
That section already holds rows for Publish and Finalize Stucks and for `Webster-Burler verify gate exhausted`, so these halts are in scope.
The record covers the two `darn.yaml` refusals properly (D7, with acceptance criteria at lines 277-279), but nowhere else:

- **Spent `Darn` verify budget.** D3 and D4 halt the run blocked at `Darn` with "the gate's reason".
  This is the darn counterpart of the existing `Webster-Burler verify gate exhausted` row.
  D4 makes `lyx loom resume` the route forward with a fresh budget, but no Decision says the Stuck reason names that route, and no row is listed.
- **D9's commit failure.** D9 names a way forward (`lyx fabric commit` in the parent pair, then `lyx loom resume`), and the acceptance criterion at line 290 reaches it.
  It still has no `refusal-spec.md` row.
- **Publish's verify-failed message.** D6 changes this message's goto target to `Darn` in a darn run.
  The existing `Publish: verify failed` row quotes the clause verbatim, so its Way forward cell has to change too, per the spec's rule that the cell and the message never disagree.
  The record doesn't say so.
- **Gate-slot sites.** The `gate slot: gate.yaml absent or invalid` row lists the sites that take a gate slot.
  The darn gate runs through `verifytree.Verify`, which takes a slot, so it becomes a new site the row doesn't name.

Line 302 then settles the spec change as "the two new rows", which pins this gap into the acceptance criteria.
The whole issue comes from the method: the record lists refusals by new config keys, not by every halt the design adds or rewords.

**Fix:** In D7, or in a refusal paragraph of its own, list every loom-section row the change adds or edits.
That means a `Darn verify gate exhausted` row whose way forward names `lyx loom resume`, with D3 or D4 stating that the Stuck reason carries that clause.
It also means a Finalize parent-records commit-failure row, the darn variant of `Publish: verify failed`'s Way forward cell, and the darn gate added to the gate-slot row's trigger.
Then rewrite the Docs acceptance criterion to name these rows instead of "the two new rows".
Add an acceptance criterion that the spent-budget halt reason names its way forward.

### [MEDIUM] A re-entered `Darn` spawn is told nothing about why it was re-entered

**Location:** `_lyx/discussion/decision-record.md:88` (D4's token list), with D6 line 124 and the stencil paragraph at line 103.

**Issue:** D4 gives the stencil exactly one re-entry input: the rejection findings, or `(none)`.
A `Darn` spawn can also start in two other ways:

- **After `lyx loom goto --to Darn` from a Publish verify failure (D6).**
  Publish's message (`internal/landingshed/verifygate.go:220`) tells the reader that "the Webster-Review round reads the Publish failure record and its gate runs the failing tests".
  D6 changes only the row name and keeps that clause, so a darn run would get a way forward that describes a row darn doesn't have.
- **After `lyx loom resume` of a spent verify budget (D4).**

Both spawns render `(none)`, and the stencil tells the agent to read the board entry and make the change.
The agent therefore starts on a branch that already holds its earlier commits, with no word that the change exists and a verify failed.
The darn gate will eventually re-prompt with the failing tests, but the first turn is spent on a task the agent treats as new.
That turn can also produce duplicate or conflicting edits, and it counts against `writer_timeout_min`.

**Fix:** Decide what the stencil gets on re-entry, for example a token naming the Publish failure record's path or the last gate failure, or a stencil sentence telling the agent to check for existing commits on the task branch first.
State it in D4, and add an acceptance-criterion row for it.
Have D6 give the darn variant of Publish's way-forward clause its own trailing text, naming the darn gate instead of a Webster-Review round.

### [LOW] `lyx loom reject` is not fully recipe-neutral for darn

**Location:** `_lyx/discussion/decision-record.md:41` (D1), with D12 at line 208.

**Issue:** D1 says `reject` needs only the recipe's `Routing`.
But `rejectVerb` (`internal/loomcli/reject.go:65-68`) checks hard-coded row names.
It accepts a run at `PR-Gate`, or a run blocked at `PR-Rework`, and its refusal message names `PR-Rework`.
In a darn run, the counterpart of "blocked at PR-Rework" is "blocked at `Darn` while holding a rejection record". The record doesn't say whether `reject` accepts that state, so a second rejection that replaces a pending one is refused there, with a message that names a row darn doesn't have.
The verb's `Long` help (`reject.go:141-150`) also says the next start "sends the findings to PR-Rework, which archives the built plan generation".
That's observable CLI text, and it is false for darn. D12 changes the board help but not this.

**Fix:** In D1, state whether `reject` accepts a darn run blocked at `Darn` with a pending rejection, and how its state check and refusal text name the rows.
In D12, add `reject`'s help to the text that names both recipes.

### [NIT] Which verbs the `verify_attempts` guard fires on is stated three ways

**Location:** `_lyx/discussion/decision-record.md:91` (D4), line 144 (D7), line 41 (D1).

**Issue:** D4 says a lowered value "halts the next step, resume or goto".
D7 says the builder refuses "at every arm".
D1 says only `step` and `run`, plus start and resume, build the shed: `goto`, `status` and `reject` read routing, and `status` and `pause` read neither.
The bound itself holds, because a gate runs only inside a built shed, and the acceptance criterion at line 279 pins `lyx shed step`.
But the record never says whether darn's `Routing` checks `verify_attempts`, so the claim that `goto` halts isn't backed by any Decision.

**Fix:** Make D4 and D7 match D1: the builder refuses on `step`, `run`, start and resume, and `goto` refuses only if darn's routing projection reads the budget, which the record then has to state.

## Focus verdicts

- **Shed builder selection at arm.** Not a defect. D1 (lines 41-44) now has loom's shared arm read the seed's recipe and choose the builder and routing for `step`, `run`, `goto`, `status`, `reject`, start and resume. It names `lyx shed step`, `lyx loom resume` and `lyx loom goto Darn` over a darn seed, and the acceptance criterion at line 265 pins them.
  The claim that the other verbs never read a recipe holds for `approve`, `pause`, `commit-records` and `validate-description`, which neither build a shed nor read routing in `internal/loomcli`. `reject` is the exception that remains, raised as F3.
- **Refusals reachable at call time.** The reader's empty-`verify` error and the builder's `verify_attempts` guard are both stated with a way forward, a spec row and acceptance criteria. The other refusals the record adds are not, so this is F1.
- **`verify_attempts` stamped at every build.** The bound holds for every path that reaches a gate (D4 line 91, acceptance criterion at line 279). How it's worded across D1, D4 and D7 is F4.
- **D10 claim clearing.** Not a defect. `markTaskDone` (`internal/landingshed/finalize.go:249-256`) only logs, and D10 names the one landed case it can clear, a merge followed by a failed done mark and a Stuck push, along with the operator's route to re-mark it.
  The halted set D10 names (`paused`, `blocked`, `failed`, `awaiting`) is exactly the set of non-`running`, non-`done` states in `internal/shedengine/status.go`. The claim that batten stays `running` over a halted child matches `innerrun.go`'s halted arm, which waits and never ends the outer run.
