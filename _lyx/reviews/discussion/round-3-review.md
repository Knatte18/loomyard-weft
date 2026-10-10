---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:165-166
    summary: The notice sender's outputs-exist stop is checked before Run.Send's idle wait, so a chair that writes its outputs during that wait meets the notice and its own gate re-prompt or pending send at the same gated-Done boundary, contrary to D7's claim that the stop keeps the sender off those boundaries
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:162
    summary: D7 serializes notices against each other and argues them off the chair's own Wait sends, but says nothing of the third pane writer D6 keeps open, lyx shuttle send on the chair's guid, which nothing serializes against the notice sender
  - id: F3
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:183-185
    summary: A resume's probe stops a finished advisor's kept strand yet the chair hears nothing, while the advisor block tells the chair a Done advisor keeps answering, so the chair's model of who is reachable is falsified silently and only the hang rule's timeout corrects it
---

### [MEDIUM] The outputs-exist stop does not keep the notice off the gate's boundary

**Location:** `_lyx/discussion/decision-record.md:165-166` (D7, Go-side notice paragraph)

**Issue:** D7 says the sender retries a line "until it lands, all the chair's outputs exist or the chair's run settles", and that "stopping once the chair's outputs exist keeps it off the boundaries where the chair's own `Wait` types, since a gate's re-prompt and pending sends go only at a gated Done, whose outputs exist".
The premise about the gate holds: in `internal/shuttleengine/wait.go` the gate's `handleGatedBoundary` is reached only from a Done arrival or from a later idle tick at that boundary, and Done needs every output file.
The stop does not deliver the claimed separation, though.
`Run.Send` (`internal/shuttleengine/run.go`, `sendVerified`) first waits in `awaitIdleSession` (`send.go`) for an idle session, up to the send-ready window, and then types; it checks nothing about output files at the moment it types.
A sender that read "outputs missing" and entered that idle wait while the chair was mid-turn sees the session go idle at the same turn end where the chair has just written its outputs.
That turn end is the gated Done boundary, where `Wait`'s `boundaryIdle` also confirms idle and the gate sends its re-prompt or pending text.
Both writers then type into the same pane at the same boundary, which is the overlap the paragraph says cannot happen.
The window is not a corner case: a chair that drafts through to its outputs in one turn, which D7's trade-off sentence names as normal, opens it on every advisor end observed during that turn.
The acceptance criteria under Running cover two notices together but no notice beside a gate send.

**Fix:** Either state a mechanism that closes the window, or state the overlap and why it is harmless.
A mechanism the design already owns: the sender re-reads the chair's outputs after the idle wait returns and immediately before typing, which shrinks the window to the gap between that read and the keystrokes, and D7 states that residue and its cost (`sendVerified`'s replay and the gate's busy/unlanded re-send both recover a garbled line).
The stronger alternative, one send lock per `Run` inside shuttle shared by `Run.Send` and the gate's `sendWithin`, is a shuttle change and would need a Scope line and a D10-style bound.
Add an acceptance criterion for an advisor ending while the chair's last turn writes its outputs.

### [LOW] `lyx shuttle send` is a pane writer D7's serialization leaves out

**Location:** `_lyx/discussion/decision-record.md:162` (D7), with `_lyx/discussion/decision-record.md:129` (D6)

**Issue:** D6 keeps `lyx shuttle send <guid>` as "the typed path for Go and the operator" into the chair.
`Runner.Send` (`internal/shuttleengine/run.go`) runs the same `sendVerified` as `Run.Send`, in another process, with no lock against an in-process sender; the support log's rejection of one goroutine per notice confirms "nothing in shuttle serializes sends on one run".
D7's serialization sentence covers notice against notice, and its stop rule argues notice against the chair's own `Wait`, but no sentence covers notice against an operator's or orch's send, which can reach the same idle boundary.
This is the same class of overlap a gate send and an operator send already have for a single agent today, so the residue may well be acceptable, but D7 neither states it nor says why it is harmless.

**Fix:** Add one sentence to D7 stating that an operator or Go `lyx shuttle send` is not serialized against the notice sender, the same as against a single agent's gate sends today, and what recovers a collision; or list it under `## Open risks`.

### [LOW] A resume silently removes a finished advisor the chair believes reachable

**Location:** `_lyx/discussion/decision-record.md:183-185` (D8), with `_lyx/discussion/decision-record.md:85` (D3, advisor block) and `_lyx/discussion/decision-record.md:178` (D7 rationale)

**Issue:** Within a call, a Done advisor keeps its strand and the advisor block tells it to keep answering after writing its output, so the chair is entitled to treat an advisor whose file exists as reachable.
On resume, D8's removing probe stops that strand (confirmed in `internal/shuttleengine/attach.go`: a terminal record over a live strand is `verdictRespawnEligible`, and `removeSupersededStrands` stops it), and D8 then classes the advisor as finished: "the chair hears nothing of it".
The chair's belief is now false with nothing to correct it, and a question it sends costs the hang rule's "reasonable time" before it writes the advisor off.
D7's rationale says "every end after a start takes one path", the queued notice; this end, caused by the engine itself, takes none.
Telling the chair costs one line on the path D8 already uses for ended advisors.
The decision is stated and its consequence is in `## Open risks`, so this is a cheap gap rather than a wrong design.

**Fix:** Have D8 send the same one-line notice for a finished advisor the probe stopped, worded so the chair knows its output stands and the seat is gone, and adjust the Probe-and-resume criterion that says it "draws no notice"; or keep the silence and give D8 the reason a notice is worse than the hang-rule wait.

## Focus verdicts

Focus 1, resume of a Done advisor: D5's two liveness sentences and D8 agree.
Within a call, live means reed tracks the strand; across a resume, the probe stops every terminal-record advisor, so what stops a kept Done advisor's strand on resume is the probe itself, before the chair's terminal result, and `Run.Stop`/`stopRecorded` keep a terminal record's outcome as D5 and D8 claim (`internal/shuttleengine/stop.go`).
The Probe-and-resume acceptance criteria cover both the Done and the timed-out case.
Whether the chair is told: no, which is F3.

Focus 2, advisor ends before the chair's handle exists: D7 settles it.
Each advisor's `Wait` join starts at that advisor's start; an end observed before the chair's handle exists is queued; the sender starts on the chair's handle, from the start or from the resume probe; a chair start failure drops the queue.
Every notice has a recipient and a path, and the criterion "an advisor that ends while the chair is still starting" covers it.
Not a defect.

Focus 3, concurrent senders: two notices cannot type at once, through the one sender goroutine, and the criterion for two near-simultaneous ends covers it.
The notice beside the chair's own `Wait` sends is F1; a notice beside an external `lyx shuttle send` is F2.

Focus 4, chair-to-advisor authority: D6 states what a chair message can do (an operator-grade instruction within the advisor's role), what bounds it (the advisor block's write confinement, prompt-enforced and said so, the agent settings' denies, no interactivity), and accepts the spawned-run wording explicitly, with the rejected variant in the support log.
The attack-surface question is answered for this edge.
Not a defect.
