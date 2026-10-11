---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D2
    summary: The directive's write-scope extension is contradicted later in the same rendered prompt by the host template's unchanged What you may write section, which says the two files are the only ones the writer may create
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D3, D4
    summary: D3's marker list omits the host's self-check command and the host steps D2 points at, so D4's lyx loom validate-discussion and Step 5/Step 6 pointers end up in the directive's own text, against D3's no-Discussion-text claim
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D6
    summary: D6's bound covers only what lands in the record, and its claim that the edge lets the writer start forks and nothing else is false, because the fork-only hook also admits a fork spawned from inside a fork
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D4, D7
    summary: A writer relaunched by reed resume after its process died is neither a fresh spawn nor an attach to a live writer; its forks are gone, and no Decision says what it does about the lens review or the marker
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md Constraints (PATTERN-daemon-wakeups), D9
    summary: The Constraints justification for keeping a 200 ms stat poll misreads PATTERN-daemon-wakeups, whose exemption covers bounded waits under a minute, not a 250 s cap, and it ignores the fswatch single-event-source rule
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D9
    summary: D9 carries support-log material, an operator attribution with a date and an enumerated list of out-of-scope future callers, where the plan writer needs only the contract
  - id: F7
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md Acceptance criteria
    summary: No criterion pins D3's reusability claim (the directive stencil names no Discussion file, section or verb) or D2's extension of the write scope by the marker alone
---

### [MEDIUM] The marker write contradicts the host template's write scope

**Location:** `decision-record.md` D2.

**Issue:** D2 places the twill block between Step 4 and Step 5 and has it extend the template's "What you may write" list by the ready marker.
The host template (`loom-template-discussion.md`) ends with "What you may write", which says "The only files you create or modify in this session are the two named in Step 5" and calls everything else read-only.
D2 also requires `single`'s rendered prompt to stay byte-identical, so that host text can't change unconditionally.
The rendered twill prompt therefore gives two conflicting instructions, and the absolute one comes last.
A writer that obeys the later section never writes the marker.
Every fork then waits out its 15 caps, about an hour, and returns "no record to review", which silently empties the lens review.
D4 step 7 does say to name such forks, but only after that hour has passed.

**Fix:** Settle in D2 how the host's list admits the marker.
One way is a second optional marker inside the host's "What you may write" list, filled with the marker path for `twill` and with nothing for `single`.
That keeps `single` byte-identical.
Add an Acceptance criterion pinning that the rendered twill prompt lists the marker among the writable files.

### [MEDIUM] D3's marker list doesn't cover what D2 and D4 make host-specific

**Location:** `decision-record.md` D3 and D4.

**Issue:** D3 says the directive's own text names no Discussion file, section or verb, and lists its markers: the files the forks review, the marker path, the rubric, the lenses and the disposition location.
D2 has the directive point at "the host template's Step 5 and Step 6".
D4 step 3 names the self-check `lyx loom validate-discussion`, and step 3 also says "both output files".
Neither the self-check command nor the host's step references are in D3's marker list.
As written, the plan writer must either hardcode Discussion's step numbers and verb into the reusable stencil, breaking D3, or invent markers no Decision names.
The rubric name in D4 step 1 is fine: D3 lists it as a marker.

**Fix:** Add the host's self-check command and its format and self-check pointer to D3's marker list, as one marker or two.
Phrase D4 step 3 in host-neutral terms, with the Discussion values given as the rendered values.

### [MEDIUM] D6's bound covers only the record, and the edge is wider than stated

**Location:** `decision-record.md` D6.

**Issue:** The bound says that whatever lands in the record still passes `validate-discussion`, parent review and Discussion-Review.
Those gates read the two discussion files only.
A fork that writes elsewhere, for example under `_lyx/config/` or in a source file, is caught by none of them.
The writer has the same stencil-level exposure, but D6 adds up to six more sessions with it, and those sessions start with the writer's full context.
The bound doesn't state what limits that.
Separately, the bound says the edge "lets the writer start in-session forks and nothing else".
The installed hook (`claudeengine/settings.go`, the `"subagent_type":"fork"` grep) admits a fork spawn from any session in the run, a fork included.
So a fork's "no Agent call" rule (D5) is stencil-level too, which the bound doesn't say.
Shuttle already collects `ForkAudit` (parent writes, spawn calls, per-fork reports) for a fork-mode run, so a cheap Go-side check exists if one is wanted.

**Fix:** Restate the bound so it covers writes outside the record and nested fork spawns.
Either name the gate or audit that catches them, or state plainly that they are bounded at stencil level only, as the writer's own scope is.
Correct "nothing else" to match the hook.

### [MEDIUM] A relaunched writer whose forks died is undefined

**Location:** `decision-record.md` D4 ("one fork pass per writer spawn") and D7 ("fresh spawn" vs "attach to a live writer").

**Issue:** Reed's resume relaunches a gated run's strand without typing a prompt (`PATTERN-completion-signal`, "The done-when list").
That covers a writer whose process died in the middle of the run.
Its background forks died with it.
The relaunched writer is not a fresh spawn, so under D7 the marker may survive.
It is also not an attach to a live writer, and under D4 it may not fork again.
If the writer was between steps 2 and 5, it waits on forks that no longer exist.
It then ends its turn with nothing outstanding, which shuttle counts as an arrival, and the run delivers a record with no lens review and no step-7 line naming the missing forks.
D7's own "either unique path or Go removal" choice can be left to the plan writer, since both options meet the staleness criterion.
This relaunch case is the gap that choice doesn't close.

**Fix:** Add to D4 and D7 what a relaunched writer does.
For example, it records the lost forks as "no report" in step 7 and finalises without re-forking.
Add a matching Acceptance criterion.

### [MEDIUM] The poll's PATTERN justification misreads the rule

**Location:** `decision-record.md` Constraints, the `PATTERN-daemon-wakeups` bullet, and D9.

**Issue:** The bullet keeps the 200 ms poll because "each await call is bounded by its cap".
The rule's background exempts "a bounded wait that ends within a minute".
The default cap is 250 s, and a fork repeats it up to 15 times.
The same background, and `PATTERN-leaf-packages` under `internal/fswatch`, make `internal/fswatch` the one file-event source that "every lyx process that waits on a file event goes through".
The verb this record generalises for every agent is exactly such a wait.
`await-review` is a precedent, but the record turns it into a shared primitive and so needs a sound basis, not the cited one.

**Fix:** In D9 and the Constraints bullet, either build the wait on `internal/fswatch` with a stat fallback at the one-second floor or above, or state the actual exemption basis and its bound.
If it is an exemption, name the `//lyx:one-shot` annotation or a guard-list entry the plan must land.

### [LOW] D9 carries deliberation and an enumerated consumer list

**Location:** `decision-record.md` D9, first paragraph.

**Issue:** "(operator, via `ly:orch`, 2026-10-11)" is provenance, and the support log already records it (Interview turn 10, Q15).
The list of future callers (seats chair under `burler-multi-llm`, the parent-review writer) names out-of-scope tasks that Scope's Out list already covers.
The requirement the plan writer needs is the stencil-facing neutrality, which the second sentence already states.
Under the completeness-before-leanness test, that requirement stays.
Only the attribution and the caller list move.

**Fix:** Keep "names no Twill, Discussion or loom file" in D9.
Drop the attribution, and replace the caller enumeration with "any agent waiting on a file".

### [LOW] Two stated behaviours lack a criterion

**Location:** `decision-record.md` Acceptance criteria.

**Issue:** D3's reusability claim, that the directive stencil names no Discussion file, section or verb, is the property Plan-Write reuse depends on, and no criterion pins it.
The same holds for D2's "extends the write list by the marker, and only by it".
The other behaviour-carrying Decisions the focus names do have pinnable criteria: D1's validation, D6's authorization, D7's staleness and D8's fan failure.
I found no criterion that pins a behaviour no Decision states.
The forks-outstanding criterion matches D4 step 5.

**Fix:** Add one criterion: the unrendered directive stencil contains no Discussion-specific path, section or verb.
Add a second: the rendered twill prompt's write scope adds only the marker path.

## Focus verdicts not raised as findings

- D6 and D9 attack surface: D9's bound ("only stats the told paths; each call ends by its cap") is sufficient. D2's marker write is bounded to the marker alone, and its defect is F1. D7's Go removal touches only the marker path and needs no further bound. D6 is F3.
- Fork premises: D4 step 5 relies on shuttle treating a gated turn end with forks outstanding as a wait. `expiredTurnEnd` in `internal/shuttleengine/wait.go` does exactly that, with `BackgroundFork` in the outstanding list. The record lists the external Claude Code dependency under Open risks, and its criterion hedges to existing coverage, so the parts are consistent. The claim that forks inherit denies holds, since the PreToolUse hooks fire inside forks; the webster guard depends on this.
- D4 ordering: the marker is written only after the self-check passes, and no fork pass follows a later re-prompt. So no fork reviews a record that is still being written, and a stale marker only matters across spawns, which leads to F4. A writer that edits the record during step 6 is the intended merge.
- D9 consistency: the cap (250 s), the Bash timeout (300000 ms), exit 0 on both outcomes and the 15-cap give-up (about 62 minutes) agree across D5, D9, Constraints and the criteria. `await-review` keeps its own `marker` output key over a shared poll, which is consistent.
- Relocation: Constraints, Auto-mode assumptions and Open risks carry requirements or recorded picks, not narrative. The Rationale paragraphs are short and decision-bearing. The only relocation finding is F6.
