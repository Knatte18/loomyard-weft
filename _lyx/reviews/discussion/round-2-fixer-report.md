# Round 2 fixer report — discussion

Every finding was validated against `internal/shuttleengine` and fixed; `lyx loom validate-discussion` passes over the edited pair.

## Fixed

- **F1 (BLOCKING, design).** Confirmed: `dispositionCandidate` in `internal/shuttleengine/attach.go` classes a tracked, live strand whose record is terminal as respawn-eligible, and the removing probe stops it via `removeSupersededStrands`.
  D5 now defines live per call and across a resume: across a resume only an advisor whose record reads running is live.
  D8 states that the probe stops a terminal advisor's kept strand, keeping its outcome, and classes an unfound advisor by its outputs: finished draws no notice, ended draws one, and a pre-interruption notice may repeat.
  D8's rationale says why the removing probe stays: shuttle hands back no handle for a terminal record, so this is the one path that leaves no strand unstoppable.
  Added a Probe-and-resume acceptance row for a Done and a timed-out advisor, scoped the Running row on kept strands to one call, and added an open risk.
  Support log: two rejected alternatives (a handle-returning probe form; probing without removal).
- **F2 (MEDIUM, design).** D7 now says each advisor's `Wait` is joined from its own start, and an advisor end seen before the chair's handle exists is queued and sent once `StartGated` returns the handle; a chair that fails to start drops the queue.
  D7's rationale says why such an end is not added to `failed_advisors`.
  Added a Running acceptance row; support log records the rejected `failed_advisors` route.
- **F3 (MEDIUM, design).** Confirmed: `sendVerified` takes no lock.
  D7 now names one sender goroutine per chair draining a queue one line at a time.
  The sender stops once all the chair's outputs exist, as well as on landing or settle; `Wait`'s own gate sends (`sendWithin` from the pending and re-prompt paths) go only at a gated Done, whose outputs exist, so the two never share a boundary.
  The chair's skill-load and prompt sends run inside `start`, before the handle exists.
  Added acceptance rows for two near-simultaneous ends and the outputs-exist stop; support log records the rejected per-notice goroutine.
- **F4 (MEDIUM, design).** D6 gains a bound on the chair-to-advisor edge: what a chair message can let through (an operator-grade instruction within the advisor's role, including relayed advice), and what bounds it (writes confined to the advisor's own outputs by its block, settings' denies, never interactive), noted as prompt-enforced.
  The spawned-run clause is accepted as rendered, with the reason.
  D3's `seat-directive-advisor` description now names the chair's role and the advisor's write limit.
  Added an open risk; support log records the rejected second directive variant.

## Disputed

None.

## Deferred

None.
