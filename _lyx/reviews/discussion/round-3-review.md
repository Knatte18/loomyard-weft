---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D3, re-prompt path and the paragraph naming what D3 replaces)
    summary: D3 replaces only the directive's two relaunch paragraphs, leaving its sentence that a gate re-prompt, a parent-review reject or a later turn of the same session never spawns forks again, which contradicts D3's re-prompt rule at both hosts
    fix: In D3, after the sentence "These paths replace the twill directive's two relaunch paragraphs ...", add that the re-prompt rule also replaces the directive's sentence "A gate re-prompt, a parent-review reject or a later turn of the same session never spawns forks again." for every host, Discussion-Write included, and that a reed resume of the same conversation is governed by the resumed paths (or, at a host with no writer fork, by the two relaunch paragraphs), not by that sentence. In Acceptance criteria, add to the Plan-Write item and to the Discussion-Write item that the rendered prompt does not carry that sentence.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md (Acceptance criteria, first item, the D3 failure-path sub-item)
    summary: The D3 acceptance sub-item names the re-prompt path only as a re-prompted master that has not made its lens spawn call, so the rule's other parts (no fork0, read the plan first if unread, no lens forks once the spawn call was made) are not pinned
    fix: In the Acceptance criteria's D3 sub-item, replace "and a re-prompted master that has not made its lens spawn call" with "and a re-prompted master, which spawns no fork0, reads the plan per D2 step 4 first if it has not, runs the lens review once if it has not made its lens spawn call, and spawns no lens forks if it has".
---

### [MEDIUM] D3's re-prompt rule contradicts a directive sentence D3 leaves in place

**Location:** `_lyx/discussion/decision-record.md`, D3.

**Issue:** The directive (`contracts/stencils/twill/twill-directive.md:20-21`) says "One fork pass per writer spawn. A gate re-prompt, a parent-review reject or a later turn of the same session never spawns forks again."
D3 states that its paths replace only the two relaunch paragraphs (lines 23-29), and the Discussion-Write acceptance item keeps those paragraphs.
Nothing says line 21 goes.
D3's re-prompt rule, which it calls host-neutral and applies at Discussion-Write too, says a re-prompted master that has not made its lens spawn call runs the lens review once.
At Plan-Write such a master has already spawned fork0, so line 21 read literally forbids the lens spawn that D3's rationale says is needed to keep the gate from passing a lens review that ran no fork.
A reed resume of the same conversation is also "a later turn of the same session", so line 21 competes with D3's resumed paths, which allow a fork0 or a lens spawn the session had not yet made.
The rendered prompt would carry two rules that disagree, and the plan writer gets no decision on which text survives.

### [LOW] The D3 acceptance sub-item narrows the re-prompt path

**Location:** `_lyx/discussion/decision-record.md`, Acceptance criteria, first item.

**Issue:** The sub-item says each D3 failure path is pinned as its own rule, but it names the re-prompt path only as "a re-prompted master that has not made its lens spawn call".
D3's re-prompt rule also says the master spawns no fork0, reads the plan first per D2 step 4 if it has not, and spawns no lens forks once it has made its lens spawn call.
The last branch is the common case, a gate re-prompt on plan findings after the lens review, and no criterion pins it.
The four resumed paths are each listed, which settles the focus's second question for resume paths.

## Focus verdicts

- D3 resume-point coverage: not a defect.
  I confirmed the premise in code: `shuttleengine/run.go` gives a gated run's strand no `DoneWhen`, and `reedengine/lifecycle.go` `planResumeLaunches` relaunches every non-live strand whose `DoneWhen` is empty or incomplete through its stored `ResumeCmd`.
  D3 states an outcome at each point: before the fork0 call (run on from the first unfinished step, fork0 included), while fork0 is in flight (no second fork0, read every plan file, complete and self-check, lens review once), after fork0 returned and before the lens spawn call (finish D2 step 4, lens review once), and at or after the lens spawn call (judge with the findings held, record the cut-short review).
  The path for a session that ends after the overview exists sends a later resume to the matching resumed path, so no point is left without an instruction.
  The only conflicting text is the directive sentence in F1.
- Acceptance criteria and the resume paths: each of D3's four resumed paths is its own listed item; the re-prompt gap is F2.
- Support log on a reed resume: Q4 says a resume of the same conversation is not a new master, keeps its per-session bound and takes D3's resumed paths, and Turn 3's scenarios line says a relaunch can also be a reed resume covered by the resumed paths.
  Both agree with D3 and its Bound line.
