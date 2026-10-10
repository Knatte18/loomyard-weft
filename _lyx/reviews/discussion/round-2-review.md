---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D5, D8
    summary: The resume probe D8 names removes the strand of a Done advisor kept by KeepPane and answers not found, so on resume a finished advisor is stopped and reported dead to the chair, contradicting D5's live definition and the Goal's ask-and-answer channel
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D7, D8
    summary: On a fresh start an advisor can die or time out after its own start and before the chair's handle exists, and no Decision says when advisor joins begin or where such a notice goes, since failed_advisors carries start failures only
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D7
    summary: One goroutine per notice puts several unserialized senders on the chair's pane, beside Wait's own gate re-prompt and pending sends, and Run.Send takes no lock, so two sends can pass the idle check at one boundary and interleave their typing
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D6
    summary: Rendering the parent directive with the chair as parent gives every chair message to an advisor operator-delegate authority and tells the advisor the chair spawned this worktree's run, and D6 states no bound on that chair-to-advisor edge
---

### [BLOCKING] Resume probe stops a Done advisor and reports it dead

**Location:** `_lyx/discussion/decision-record.md` D5 and D8; acceptance criteria under Probe and resume.

**Issue:** D5 defines an advisor as live while reed still tracks its strand, and keeps a Done advisor reachable through `KeepPane`.
D8 probes each advisor "with the same probe shape ungated" as the chair's superseded-removing probe.
That probe does not share D5's meaning of live.
`finalize` in `internal/shuttleengine/wait.go` writes the Done outcome into the advisor's `run.json` and, with `KeepPane`, leaves its strand alive.
On the next call, `dispositionCandidate` in `internal/shuttleengine/attach.go` classifies a tracked, live strand whose record carries a terminal outcome as `verdictRespawnEligible`, never attachable.
The probe then answers not found, and because removal of superseded strands is on, `removeSupersededStrands` stops that strand first.
The same holds for a timed-out advisor whose pane is still up.
So after any resume with the chair live, every advisor that had already written its output is killed by the probe and falls into D8's "dead or absent" branch, and the chair gets a death notice for an advisor that finished.
The chair loses the advisor the Goal says it can keep asking, and the Probe-and-resume acceptance criterion "a dead advisor is not restarted and the chair is told once" fires for a seat that is not dead.
A probe without removal (`AttachIfLive`'s shape) does not help: it still answers not found for a terminal record and hands back no handle, so the strand outlives the chair's terminal result with nothing to `Stop` it.

**Fix:** State in D8 how a resume recognises an advisor whose record is terminal and whose strand is live, and what it does with it: either reach it without the removing probe and keep a way to stop it at the chair's terminal result (for example, stopping it by guid through shuttle's stop verb), or decide that a resume treats a finished advisor as finished, not dead, and send no notice for it.
Align D5's definition of live with whatever D8 decides, and add a resume row for a Done advisor to the acceptance criteria.

### [MEDIUM] A notice raised before the chair has a handle has no recipient

**Location:** `_lyx/discussion/decision-record.md` D7 and D8.

**Issue:** Advisors start first and the chair last (D8), and the chair's `StartGated` blocks through its provider's startup window.
D7 sends the death or timeout notice "by `Run.Send` on the chair's handle", and `failed_advisors` carries start failures only.
No Decision says when an advisor's `Wait` join begins.
If it begins when the advisor starts, as burler's halves do, an advisor that dies during the chair's startup raises a notice with no chair handle to send it on.
If it begins only after the chair is up, the advisor's death is still observed, but the record should say so, and the case of an advisor that died before the chair's prompt was filled, and so could have gone into `failed_advisors`, is undecided either way.
Every other notice in D7 has a defined recipient and path; this window does not.

**Fix:** In D7 or D8, state when the advisor joins start relative to the chair's start, and where an advisor end observed before the chair's handle exists goes: into `failed_advisors`, or held until the chair's handle exists and then sent like any other notice.

### [MEDIUM] Concurrent unserialized sends into the chair's pane

**Location:** `_lyx/discussion/decision-record.md` D7, the Go-side notice paragraph.

**Issue:** D7 names one goroutine per notice, each retrying `Run.Send` until it lands or the chair settles.
`sendVerified` in `internal/shuttleengine/run.go` waits for an idle session and then types; nothing serializes two callers on one run.
Two advisors ending close together give two goroutines that can both see the same idle boundary and type into the chair's input box at once.
The chair's own `Wait` sends into the same pane at a turn boundary too, through `handleGatedBoundary`'s re-prompt and pending sends, and those also fire at idle.
The record names what bounds each retry, the send-ready window and the chair's settle, and on that point its description of `Run.Send` matches the code.
What it leaves open is the interleaving of its own senders.

**Fix:** In D7, route every notice through one sender per chair, a single goroutine fed by a queue that sends one line at a time, and state how it avoids typing over a send from the chair's own `Wait`, or state why the overlap is harmless.
Cover two near-simultaneous advisor ends in the acceptance criteria.

### [MEDIUM] The chair-to-advisor edge carries delegate authority with no stated bound

**Location:** `_lyx/discussion/decision-record.md` D6, "An advisor's `parent_directive` is rendered with the chair's strand name as the parent name".

**Issue:** `contracts/stencils/parent/parent-directive-parent.md` says the named parent is "the session that spawned this worktree's run", and that a message from it "is the operator's delegate, and you act on it like an operator instruction, within your role's limits".
Rendered with the chair's name, each advisor treats any chair message as an operator instruction.
The chair is itself an LLM whose inputs include other advisors' messages, which D6 calls advice and never instruction.
The chair can relay what one advisor writes to another as an operator-grade instruction, so D6's rule that advisor content never carries authority holds only as far as the chair's judgment.
D6 bounds the advisor-to-parent edge but says nothing about what this new chair-to-advisor edge can let through or what bounds it, which the rubric's attack-surface test requires.
The rendered text also tells the advisor a falsehood: the chair did not spawn the worktree's run.
On the Goal's claim: an advisor's `SendMessage` route goes to the chair alone, so the channel stays outside the parent as far as advisors obey their prompts, which D6's existing bound already states.

**Fix:** In D6, state what an advisor may do on the chair's word and what bounds it: the advisor's own write surface, its outputs, and its settings' denies.
Either accept the parent-directive wording, including the spawned-run clause, with that bound stated, or carry the chair's role in the advisor block and render the advisor's `parent_directive` some other way.

## Focus verdicts

- D7's description of `Run.Send` matches `internal/shuttleengine/run.go` and `send.go`: idle wait, `ErrSessionBusy` past the window, and a notice that lands only at a turn boundary the chair ends without its outputs.
  D7 owns the conflict with D6's ask-and-keep-drafting rationale as an accepted trade-off and lists it under Open risks, so that conflict is not a defect.
  The remaining gap is F3.
- An advisor that timed out or died before the chair, within one process, still has its in-process handle.
  `Run.Stop` keeps its terminal outcome and removes its strand if live, so D5's rule stops it correctly.
  Across a resume it is caught by F1.
- Once an advisor writes its output, `KeepPane` keeps its strand and run directory, as D5 says, and the chair can still reach it within one call.
  Across a resume it cannot, which is F1.
