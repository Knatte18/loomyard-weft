```yaml
lens: fix-way-forward
outcome: reported
```

Restated task: check that each refusal, notice, halt and retry this fix introduced can be carried out from the state it names and leads to progress.

Ways forward the fix introduced:
1. The D9 Finalize halt names `lyx batten cancel <member> --remove` for each member, then `lyx batten run <slug>`.
2. The D9 interrupted-landing notice names `lyx beam start <slug>` and `lyx beam land <slug>`.
3. On a failed land check, D9 restarts the dobby, removes the request and keeps waiting.
4. On a failed status write, D9 keeps the request and the row, and the next wake retries.
5. D6's adopt skip reports the beam naming `lyx batten status <slug>`.
6. The new acceptance criteria: F1, F2, the F4 race, F5's adopt orders, the adopt skip, and F17's items.

Findings:

- **MEDIUM**, D9's new Finalize bullet ("the halt names `lyx batten cancel <member> --remove` for each, then `lyx batten run <slug>`"). Following the named steps leads to a second halt, not progress.
  `lyx batten cancel --remove` removes the pair and settles the run claim, but the entry stays open with its `project` on the beam (`internal/pairteardown/doc.go:63`, "settles the board entry").
  `BeamLandRefusal` refuses an open member that has no pair, with its own way forward: "move its `project` off the beam with `lyx board upsert`, or land it where it runs" (`internal/battencli/beamland.go:83`).
  So after the cancel, `lyx batten run <slug>` re-enters Finalize and halts again.
  The halt should carry `BeamLandRefusal`'s own per-member ways forward, both the has-pair line and the no-pair line, rather than a fixed cancel-then-run sequence.
- **MEDIUM**, D9's Finalize bullet ("so a member a hand `lyx batten run` started between Beam-Await's re-check and its status write never lands under the beam"), and the matching acceptance criterion. The halt is not reached when the member has already finished.
  A late member forked from the beam can run to its own Land into the beam worktree while the beam is in Parent-Align or the perch.
  Its teardown then leaves no open entry and no pair, so `BeamLandRefusal` passes at Finalize and the beam lands that member's commits, which the perch never reviewed.
  The halt only catches a member that is still open, so the "never lands under the beam" claim does not hold for this state.
- **LOW**, D9's interrupted-landing notice ("naming `lyx beam start <slug>` and `lyx beam land <slug>`"). If the crash came after Beam-Await's status write naming Parent-Align, the re-entered row's status write is warn-only (`internal/loomcli/wiring.go:226-235`).
  If that write fails, the status stays at Parent-Align, the beam holds the landing hold, and `lyx beam start <slug>` is refused by D3's landing-hold refusal.
  `lyx beam land` is refused too, because the run does not read as at Beam-Await.
  The named steps are then unreachable until the next status write succeeds, and the notice says nothing of this.
- **LOW**, D6's adopt skip ("reports that beam naming `lyx batten status <slug>`, and still writes its owner"). When this machine does not also own the beam's parent, writing the owner pauses the landing on the old machine, because the pawl run's verdict reads the beam's owner (D11).
  D6's resume rule then does not resume the run here, so nothing drives it.
  `lyx batten status` only shows the pause; the step that moves it forward, `lyx adopt <parent>`, is not named.

Tool calls:
- Grepped `internal/pairteardown/doc.go` and `BeamLandRefusal` in `internal/battencli/beamland.go`. My context lacked what `cancel --remove` does to the board entry and the refusal's per-member wording.
