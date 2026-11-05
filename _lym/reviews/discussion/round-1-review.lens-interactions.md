```yaml
lens: interactions
outcome: reported
```

**Pairs checked**
- D2 × D10 (ready record standing after goto, dobby's start hold): held.
- D2 × D11 (hold named Finalize, entry check on re-entry): held.
- D2 × D13 (move removes the ready record, lifting the start hold): held.
- D3 × D5/D6 (ready needs a stamped description; verbs refuse after ready): held.
- D3 × D2 (open-work refusal before ready, start hold after): held.
- D3 × D10 (goto after ready lands again without D3's checks; Finalize's merge-in and halts cover it): held.
- D3 × D13 (D3's "Finalize's landing and Beam-Down end it" vs D13 removing the ready record): broken, F4.
- D4 × D6 (verbs write and commit the record): held.
- D4 × D8 (Finalize drops PR after parent-side landing; crash re-enters via already-landed): held.
- D4 × D13 (D13 carries the PR into D4 and removes the tracked ready record): broken, F2.
- D4 × D13 (carried record has no description stamp → D3 refusal 4 asks for describe; notice names the new verbs): held.
- D6 `close` × D8 bullet 2 (close for "land without one" vs Finalize opening one for `merge`): broken, F3.
- D6 `open` reopen × D8 closed-PR halt (verb refuses past ready, halt names the operator on GitHub): held.
- D6 × D2/D10 (verbs refuse while the ready record stands, goto included): held.
- D7 × D4/D6/D13 (announce head set by `open` and the carry; `update` leaves it): held.
- D7 × D8/D12 (dobby down from the accept on; `beam start` in landing leaves it down): held.
- D8 × D3 refusal 5 (required with none recorded refuses ready, so D8 bullet 2 is reached only by `merge`, a rule change or a goto): held.
- D8 × D9 (squash text from description; found PR text never edited): held.
- D10 × D13 (PR-Review retired, reroute removed; old-binary reopen request): broken, F5.
- D11 × D2's accept removal order (crash "between the accept's removals and Done"): broken, F1.
- D11 × D13 (retired-name entry check; D13 lifts the board hold and removes the marker, notice names `beam start`): held.
- D12 × D2 (`beam start` before the accept starts both; in landing the orch only): held.
- D13 × D5/D6 ("halted" counts as Beam-Await running or halted for the verbs): held.
- Acceptance criteria D1–D14 vs their decisions: consistent except D11's and D13's criteria inherit F1/F2.
- Constraints (remote-refs lock, tracked-file locks, stop chain) vs D3, D4, D6, D7, D13: consistent.

**Findings**

F1 — MEDIUM — design — decision-record.md:150 (D11) with D2's accept
D11 says a crash "between the accept's removals and Done" is recovered by starting the dobby when the dobby-stop marker stands, but the accept's removals include that very marker, so in exactly that window the marker is gone: the dobby stays down, no land request is ever rewritten, and the run waits for a hand `lym beam start` — contradicting D11's no-hand-step purpose.
Evidence: internal/pawlengine/beamawait.go:306-310 (accept removes LandRequest, DobbyStopMarker, AwaitMarker before Done); beamawait.go:190-195 (interrupted entry today only removes the marker and notifies).

F2 — MEDIUM — consistency — decision-record.md:168-169 (D13) vs decision-record.md:75-76 (D4)
D13 writes D4's tracked landing record (the carry) and removes the tracked ready record but states no commit, while D4 names its writers as committing ("write and commit it", "drops … committed") and the existing ready-record removal commits the landing directory; a moved run leaves uncommitted tracked changes in the beam worktree.
Evidence: decision-record.md:75-76, 168-169; internal/beamcli/doc.go:52 (RemoveReady then commits the landing directory).

F3 — LOW — consistency — decision-record.md:102 (D6 close) vs decision-record.md:126 (D8 bullet 2)
`close` comments on GitHub that the operator chose to land without a pull request, but on a beam.yaml `merge` parent D8 bullet 2 still finds or opens a pull request and merges it on GitHub, so the comment is false and the orch's skip is silently overridden; neither decision states this case.
Evidence: decision-record.md:102, 126.

F4 — LOW — consistency — decision-record.md:61 (D3) vs decision-record.md:169 (D13)
D3 says only Finalize's landing and Beam-Down end the ready record; D13's move also removes it.
Evidence: decision-record.md:61, 169.

F5 — LOW — design — decision-record.md:165-170 (D13) with decision-record.md:142 (D10)
D10 removes Beam-Await's reopen handling, which today removes a reopen request on entry; D13's move removes the ready record, land request and dobby-stop marker but not a reopen request the old binary's `lym beam reopen` left for a run at PR-Review, so that file is left with nothing that reads or removes it.
Evidence: internal/pawlengine/beamawait.go:213-248 (answerReopen); decision-record.md:142, 169.

Tool calls: none (inherited context held every definition used).
