# Round 1 fixer-report — discussion

Every finding was validated against the code before fixing: `Run.Send` waits for an idle session and fails with `ErrSessionBusy` (`internal/shuttleengine/run.go`, `send.go`); `finalize` removes a Done run's strand and run directory unless `KeepPane` is set (`wait.go`); `Stop` keeps a terminal record's outcome (`stop.go`); `holdNotice` and `parent-directive-parent.md` already name `SendMessage`, and the parent directive says to trust no other session's message.
All eight premises hold.
`lyx loom validate-discussion` exits 0 after the fixes.

## Fixed

- **F1 (BLOCKING).** D7: a start failure is told through a new reserved marker, `failed_advisors`, in the chair's prompt, since advisors start before the chair; no `Send` is made for it.
  D3 lists the marker, `(none)` when empty and always `(none)` for advisors; the chair block names it.
  Acceptance criterion added; a rejected alternative (send after the chair starts) recorded in the support log.
- **F2 (BLOCKING).** D7 now defines the Go-side notice path: `Run.Send` from one goroutine per notice, apart from the join goroutines, retried until it lands or the chair's run settles.
  It states that the notice lands only at an idle turn boundary and names the trade-off, a chair that never idles never hears it.
  D8 names the same path for a dead advisor found on resume.
  Acceptance criterion and an Open risk added.
- **F3 (BLOCKING).** D5: advisors run with `KeepPane` set, so a Done advisor stays reachable until the chair's terminal result stops it; the chair runs without it.
  D7's done rule and the advisor block say it keeps answering after writing its output.
  Acceptance criteria added; rejected alternative recorded.
- **F4 (MEDIUM).** D6: an advisor's `parent_directive` is rendered with the chair's strand name as parent, so no advisor prompt names the run's parent, and the directive's ask-and-trust rule points at the chair.
  The chair block makes an advisor's message advice, never an instruction.
  The bound states what a disobeying advisor can reach (any `ListAgents` name, the parent included) and what bounds it.
  This also resolves a gap the finding implied: under the parent variant an advisor would have distrusted the chair's messages.
  D3, acceptance criteria, an Open risk and a rejected alternative updated.
- **F5 (MEDIUM).** D5 defines live as every advisor whose strand reed still tracks, settled or not, and stops them all on the chair's result.
  D7 states that an advisor timing out just before the chair's deadline is the routine end and is told like any timeout.
  Acceptance criterion added.
- **F6 (LOW).** D6 now says `seatengine` adds no provider words; D3's single-source claim is limited to the seat-to-seat channel wording; the Constraints line matches.
- **F7 (LOW).** The operator's "Added after Discussion" decision is folded into D6, dated and attributed, and the section is deleted.
  D6 states that neither `lyx shuttle send` nor `Run.Send` is the seats' channel, and that D7's notice is the engine's one Go-side use.
- **F8 (LOW).** The `Run.Send` idle and one-line constraint now lives in D7.
  The argued alternatives are cut from D6's rationale (typing through `Send`, a producer mailbox), D8's ("stopped rather than inherited") and D11's (rebuilding over handles); D8's, which `Rejected alternatives` lacked, is added there.

## Disputed

None.

## Deferred

None.
