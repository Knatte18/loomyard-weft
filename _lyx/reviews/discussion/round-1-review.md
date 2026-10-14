---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:66
    summary: D3 treats every ended master session as a fresh respawn or a live re-prompt, misses reed's resume of the same session, and leaves the directive's existing relaunch rules undecided
    fix: In D3, add the resumed-session paths. First, a master resumed in the same conversation (reed Resume through the strand's ResumeCmd, for example after a hibernate) whose fork0 call was in flight has lost fork0, spawns no second fork0, reads every plan file on disk, completes the plan itself per the format steps and self-check as in the first path, then runs the lens review once. Second, a master resumed at or after its lens spawn call keeps today's cut-short rule (resume at judging with the findings it holds, and record each fork that had not returned as no report). State that D3 replaces the twill directive's two relaunch paragraphs, which key on a single Agent call that fork0 makes ambiguous. Restate the third path for a session that is no longer live, so its outcome does not depend on a gate re-prompt reaching a dead agent. Add both resumed paths to the Acceptance criteria list of D3 failure paths at decision-record.md:173.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:108
    summary: D6 bounds a dispute with code-level evidence kinds that do not fit Discussion-Write, so there the judge must either break the writer's Step 2 bound or can dispute nothing
    fix: In D6, make the dispute evidence kinds host-specific. At Plan-Write keep the current list. At Discussion-Write name the board entry, the discussion spec, the rubric, a PATTERN entry, the self-check's output and a coarse code read within Step 2's bound, with no path:line required. Deliver the kinds as a host value, like the governing input. Add an Acceptance criterion that the Discussion-Write lead prompt's judge step names those kinds. Extend the Open risk on Discussion-Write to cover this bound.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/support-log.md:72
    summary: Q4 says a relaunch respawns fork0 once only when the overview is missing, which contradicts the per-session bound and the whole-procedure respawn that D3 now states
    fix: Rewrite the Q4 pick to match D3. The master finishes an incomplete plan itself. A fresh respawn before the overview exists runs the whole procedure, fork0 included. Each master session spawns fork0 at most once, and a respawned session counts as a new master.
---

### [BLOCKING] D3's session-end paths assume the wrong recovery mechanism

**Location:** `_lyx/discussion/decision-record.md:66`

**Issue:** D3's second path says that when the master's session ends, "shuttle respawns a fresh session, whose plan-directory rotation archives the partial card files".
That is one recovery path, but not the only one.
Reed's Resume relaunches a dead strand through its stored `ResumeCmd` (`internal/reedengine/strand.go:575`, `internal/reedengine/lifecycle.go:838`).
For a Claude session that command is `--resume` on the same session id (`internal/shuttleengine/claudeengine/command.go:193`), so the conversation survives and every fork it had spawned is lost.
A gated run's strand carries no done-when list, so reed revives it even when the output files exist (`internal/shuttleengine/doc.go`, "a gated run's strand carries none").
Plan-Write is gated, and `lyx hibernate` now drives exactly this kind of pause and resume.
Today's directive covers this case in its two paragraphs, "relaunched after its process died before its Agent call" and "at or after its Agent call" (`contracts/stencils/twill/twill-directive.md`).
Under fork0 the master makes two Agent calls, so "its Agent call" no longer says which one.
D3 neither keeps, replaces nor rewrites those paragraphs.
A master resumed while fork0 was in flight therefore has no defined outcome.
D3's bound forbids a second fork0 in that session.
The first path does not apply either, because fork0 never returned.
The third path ("the gate re-prompts") also assumes a live agent.
When the session is dead, shuttle's liveness finalize evaluates the gate's Final closure instead of re-prompting (`internal/shuttleengine/doc.go`, the gate section).
This answers the first focus item.
For a live re-prompt, D3's rule is bounded: a master that has not made its lens spawn call runs the lens review once.
The resumed-session paths are the ones left unbounded.

### [MEDIUM] D6's dispute evidence does not fit Discussion-Write

**Location:** `_lyx/discussion/decision-record.md:108`

**Issue:** D6 limits a dispute's evidence to "a `path:line` or glyph, a `PATTERN` entry, a test or caller the change would break, or the self-check's output".
At Discussion-Write, the writer's own stencil, Step 2, says it "MUST NOT gather exact signatures, `file:line` citations".
There, findings judge a design against the board entry, not against code.
The record itself shows the mismatch: its two `every fix worse` disputes (`support-log.md:46` and the Auto-mode assumptions dispute after it) cite the board entry's Evidence and the discussion spec.
Neither is among D6's kinds.
So at Discussion-Write the bound is either unmeetable, and every finding must be fixed, which is the additive growth the rubric warns about, or the judge ignores it.
The Open risks note only that no bench measured the judge there.
Neither a Constraint nor an Acceptance criterion bounds it.

### [LOW] Q4 is stale against D3

**Location:** `_lyx/discussion/support-log.md:72`

**Issue:** Q4's pick reads "a relaunch respawns fork0 once only when the overview is missing".
Turn 3's scenarios finding replaced that bound with a per-master-session one.
D3 now says a fresh respawn runs the whole procedure, fork0 included, with no "once only" cap across respawns.

## Verdicts on the remaining focus items

- D1 and D6, the removed selector and rejection rule: not a defect.
  D1 bounds fork0 to runs whose lens forks are on.
  D6's dispute rule is stricter than the retired rule: every severity needs evidence, and one sound change forces a fix.
  At Plan-Write, the `every fix worse` ground is bounded by the same evidence rule.
- D2 and D4, what fork0 may read, write or run: the record's D2 holds it all (reads only what its context lacks, never the Agent tool, the lens review file or git).
  Nothing of it lives only in the support log.
- Requirement-to-criterion coverage: apart from F1's paths and F2's evidence kinds, each requirement in D3 to D9 maps to a criterion.
  The Constraint that Discussion-Write's prompt with lens forks off is byte-identical has no criterion of its own.
  No twill text renders in that case, so it is not raised.
- Relocation: no Rationale or Open-risk line is narrative without a requirement that is worth moving.
