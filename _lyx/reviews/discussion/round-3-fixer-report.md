# Round 3 fixer report — discussion

All three findings were checked against the code and fixed; `lyx loom validate-discussion` passes after the fixes.

## Fixed

- **F1 (MEDIUM, design): the outputs-exist stop does not keep the notice off the gate's boundary.**
  Confirmed: `sendVerified` (`internal/shuttleengine/run.go`) runs `awaitIdleSession` and then types, with no output check between the two, and the gate's `sendWithin` (`send.go`) runs the same sequence with no lock.
  D7 no longer claims the stop separates the two senders.
  Instead it adds one send lock per shuttle `Run`, held across the whole idle-wait, typing and confirm sequence by both `Run.Send` and `sendWithin`.
  D7 states the lock's bound: it skips no idle wait, delivery check or gate evaluation, and a send waits on it no longer than the holder's windows.
  It also says what happens to the line that goes second: the gate's existing turn-start hold (`boundaryIdle`) covers the gate, and the notice sender's retry covers the notice.
  Scope gains a line for the lock, and D14 adds `internal/shuttleengine/doc.go`.
  Running gains a criterion for an advisor that ends during the chair's last turn, and Shuttle gains one for the lock.
  The notes for the plan writer name the two send sites.
  The support log rejects the reviewer's weaker option, a re-read of the outputs before typing, because it needs a shuttle hook anyway and only narrows the window.
  It also updates the one-goroutine-per-notice rejection, whose reason had relied on nothing in shuttle serializing sends.
- **F2 (LOW, consistency): `lyx shuttle send` is a pane writer D7's serialization leaves out.**
  Confirmed: `Runner.Send` runs `sendVerified` in another process.
  D7 now states that it is outside the lock, the same as beside a single agent's gate sends today.
  A collision garbles or fails one line, and each sender's own path recovers it: the notice sender's retry, the gate's busy and unlanded re-send, or the verb's failure report.
- **F3 (LOW, design): a resume silently removes a finished advisor the chair believes reachable.**
  Confirmed: in `internal/shuttleengine/attach.go` a terminal record over a live strand is `verdictRespawnEligible`, and `removeSupersededStrands` stops it.
  With the chair live, D8 now sends one notice for every advisor the probe does not find.
  The line depends on the advisor's outputs: a finished advisor's says its output stands and the seat no longer answers, and an ended advisor's reads like a mid-step death.
  D8's rationale gives the reason.
  D5's across-resume sentence, D7's rule list, the open risk and the Probe-and-resume criterion now match.
  The support log records keeping the stop silent as a rejected alternative.

## Disputed

None.

## Deferred

None.
