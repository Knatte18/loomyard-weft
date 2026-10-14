---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D3 path list and Acceptance criteria D3 item
    summary: D3 defines no outcome for a master resumed before its fork0 call or after fork0 returned but before its lens spawn call, and its paths replace the directive paragraph that covered the first case
    fix: "In decision-record.md D3, add two paths after the fork0-in-flight path. (a) A master resumed in the same conversation before its fork0 call runs the procedure on from the step it had not finished, fork0 included, since the bound allows it one fork0. (b) A master resumed in the same conversation after fork0 returned and before its lens spawn call spawns no second fork0, finishes D2 step 4 (reads the plan files in full, self-check) if unfinished, then runs the lens review once. In the Acceptance criteria D3 item, add both cases to the list of failure paths the rendered prompt carries as their own rules."
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/support-log.md Question ledger Q4 and Interview Turn 3 scenarios line
    summary: Q4's pick and the Turn 3 note that a relaunch is a fresh spawn omit D3's resumed-conversation paths and the rule that a resumed session is not a new master
    fix: "In support-log.md Q4, extend the pick: a session resumed in the same conversation through reed's ResumeCmd is not a new master, keeps its per-session bound, never spawns fork0 or lens forks a second time, and takes D3's resumed paths. In Interview Turn 3, after the scenarios MEDIUM line on relaunch being a fresh spawn, add that a relaunch can also be a reed resume of the same conversation, which D3 covers with its resumed paths."
---

### [MEDIUM] D3 leaves two resume points without an outcome

**Location:** `_lyx/discussion/decision-record.md` D3 path list and Acceptance criteria D3 item

**Issue:** D3 states that reed resumes a gated run's strand through its stored `ResumeCmd`, verified in code: `internal/reedengine/lifecycle.go` relaunches every non-live strand whose `DoneWhen` list is empty or incomplete, and `internal/shuttleengine/doc.go` says only an ungated run's strand gets its output files as a done-when list.
A resume can therefore come at any point in the procedure, but D3 defines the outcome only for a master resumed while fork0 was in flight and for one resumed at or after its lens spawn call.
Two points are left without a stated outcome:

- Resumed before its fork0 call, during exploration or the rubric read.
  Today's directive covers this case with "A writer relaunched after its process died before its Agent call still spawns once", and D3 says its paths replace that paragraph at Plan-Write.
  The Bound sentence implies the master may still spawn fork0, since a resumed session keeps its own count of zero, but no path tells it so, and the replaced paragraph no longer does.
- Resumed after fork0 returned and before its lens spawn call, during D2 step 4.
  The re-prompt rule covers a gate re-prompt or parent-review reject and says what to do there, but a reed resume is neither, so whether it applies is left to the agent.

Because the rendered paths replace the directive's relaunch paragraphs, a missing path leaves the agent no instruction at all for that point.
The Acceptance criteria list the D3 failure paths one by one and omit these two, so nothing pins them.

The other two cases this round's focus names are settled.
The fork0-in-flight path and the at-or-after-lens-spawn path each state an outcome.
The no-longer-live path matches the code: `internal/shuttleengine/gate.go` and `wait.go` evaluate each entry once at a liveness finalize, through `Final` where one is set, and `internal/loomshed/gates.go` `ValidatePlanLensReview` raises `lens-review-missing` when the lens review file is absent.
D3 also says what happens to the directive's two relaunch paragraphs at each host, and an Acceptance criterion pins it at both hosts.

### [LOW] Support log Q4 and Turn 3 still describe relaunch as a fresh spawn only

**Location:** `_lyx/discussion/support-log.md` Question ledger Q4 and Interview Turn 3

**Issue:** Q4 asks what happens when "the master is relaunched", and its pick covers only a fresh respawn: "a fresh respawn before the overview exists runs the whole procedure … a respawned session counts as a new master".
D3 also defines a reed resume of the same conversation, and its Bound says "a resumed one does not" count as a new master.
The Q4 pick agrees with D3's per-master-session bound and whole-procedure respawn for the respawn case, but it says nothing about the resume case.
Turn 3's scenarios line records the premise "relaunch is a fresh spawn whose rotator archives the partial plan", which D3's resume paths now contradict.
The plan writer never reads the support log, so the record is not wrong because of this, but the log no longer explains the record's resume decisions to a reviewer.

## Settled without a finding

On the D6 directive, the verdict is no defect.
D6 gives Discussion-Write its own evidence kinds as a host value: the board entry, the discussion spec, the rubric, a `PATTERN` entry, the self-check's output, or a coarse code read within the writer's Step 2 bound, with no `path:line` required.
Each of these is reachable under `loom-template-discussion.md` Step 2, which allows a coarse read of module boundaries and pattern conflicts and bans only exact signatures, `file:line` citations, interface shapes and dependency lists.
The bound is therefore meetable, not silently ignored, and the Acceptance criterion "The rendered judge step names each host's dispute evidence kinds from D6" carries it, with the Open risks recording that no bench measured it.
