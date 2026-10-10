# Round 3 fixer report: discussion

## Fixed

- F1 (MEDIUM, decision): D2's Decision now says a conflict-session stop that returns an error leaves the merge untouched and stops the row Stuck, naming the strand's guid and the way forward `lyx reed remove <guid>`, then `lyx loom resume`.
  This follows the existing refusal-spec rows for strands lyx could not stop.
  D2's Bound states that the abort runs only after a successful stop, and D3's Bound covers the failed-stop case.
  The acceptance criteria add the `Publish or Finalize: conflict session not stopped` refusal-spec row and the failed-stop behavior.
  `PATTERN-no-denied-recovery`'s constraint line now lists `lyx reed remove`.
  Premise checked: `Runner.StopStrand` and `removeStrandIfLive` in `internal/shuttleengine/stop.go` return errors on a failed record write or reed probe.
- F2 (LOW, design): D3's Decision no longer says the no-live-driver refusal names `lyx loom start`; it says the refusal and its way forward.
  D3's Rationale now says a dead driver comes back through `start`, or for a halted run through batten's revive, and that the resume after a revive takes the live-driver path.
  Premise checked: `noLiveDriverMessage` in `internal/loomcli/resume.go` names `lyx batten run <slug>` for a dead strand, and `battenshed.reviveDeadDriver` revives it.
- F3 (LOW, decision): D6's Decision now says a named caller whose parent resolution errors is refused, with a way forward that names the error and the operator's unnamed shell.
  D6's Bound says an unresolvable parent never lets a named caller through, and the `decision add` acceptance criterion covers the case.
  Premise checked: `hubgeom.ResolveParent` in `internal/hubgeom/parent.go` returns errors for an unreadable origin record and for failing to tell the prime from a pair.
- Support log: the Question ledger records the auto-picks for F1 and F3.

## Disputed

None.

## Deferred

None.
