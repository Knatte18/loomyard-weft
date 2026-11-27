```yaml
lens: code-correctness
outcome: reported
```

Lens code-correctness: I ran each function, branch and error path the diff adds or changes through a concrete case.

**Per function or branch: the case run, and the result**

- **`clearRunPhase`, the `remove && beam` branch** (`internal/battencli/cancel.go:578-586`)
  - **A beam with no members, `--remove`:** RemovePair → RemoveRecords → ClearClaim → BeamProject true → EndBeam true → `BeamEnded`. Correct.
  - **EndBeam fails:** the cancel returns the error naming the `--remove` re-run, with the locks released by the deferred `hold.end`. On the re-run, `refuseNothingLeft` reaches the new beam branch (`cancel.go:779-785`) and the run finishes with EndBeam. Correct, but see finding 3 on the push that was owed.
  - **The BeamProject read fails here:** it is reported as "could not clear the beam flag". The wording is slightly off but still names the re-run (NIT, not filed).
  - **A task, `--remove`:** BeamProject is false, so EndBeam is not called. Correct.
  - **A beam without `--remove`:** the branch is skipped and the flag is kept. Correct.
- **`refuseNothingLeft`, the new beam branch** (`cancel.go:779-785`)
  - **Only the flag is left, `--remove`:** returns nil. The flow then runs TakeMarker, EndForCancel, StopBeam (a no-op, since no worktree exists), RemovePair (ErrPairNotFound → SettleTeardown), ClearClaim and EndBeam. Correct.
  - **The BeamProject read fails:** `probeRefusal` names the board entry. Correct.
  - **End-only:** cannot happen for a beam, because `refuseBeamCancel` already refuses an unreachable origin (`cancel.go:479`). Correct.
- **`cancelNotice`, the beam suffix** (`cancel.go:869-877`)
  - The suffix shows only in the `PairRemoved` case. That covers every case where `BeamEnded` is set, because `removePairAndBranch` always sets `PairRemoved` before EndBeam runs. Correct.
- **`cancelEnvelope`, `beam_ended`:** carries the field through. Correct.
- **`hubCancelDeps.StopBeam`, the new origin-mark skip** (`internal/battencli/cancelcmd.go:396-400`)
  - **No origin record (`!found`):** returns nil and skips. Correct.
  - **The record exists but `Beam` is false:** returns nil and skips. Correct for the orch, which BeamDown refuses on an unmarked pair.
  - **`ReadOrigin` fails:** returns the error, which the cancel adds as a warning. Correct.
- **`hubCancelDeps.EndBeam`:** a failure to open the board is returned. Correct.
- **`Board.EndBeam`** (`internal/boardengine/beamend.go`)
  - An absent entry, a task, a plain project or a beam already ended each give `noWrite` and false. Correct.
  - **A beam:** clears beam, recipe, owner and colour, then saves. The run-held check passes, because ClearClaim has already cleared the run status (`runlock.go:56`).
  - **The owner write:** EndBeam writes `OwnerMachine` outside `SetOwners`, through the ordinary write and its detached sync. This breaks the board's stated owner invariant (finding 1).
- **`checkBeamPayload`, the reworded message:** the `%s` placeholders and the three `slug` arguments match. Correct.
- **The `cancelFake.EndBeam` fake:** answers `beamProject && endBeamErr == nil`, which is consistent with the rows. Correct for this lens.

**Findings**

1. **MEDIUM**, class design, `internal/boardengine/beamend.go:19`. EndBeam clears a beam project's `OwnerMachine`, which makes it a second owner writer. The package states that "SetOwners is the only writer of an owner", and that it pulls the board level with its upstream, writes one commit and pushes it (`internal/boardengine/doc.go:80-81`; `doc.go:76` also says an owner change is never a scope change). EndBeam instead writes through `boardCriticalSection` with a detached sync, with no pull first. A concurrent owner write on another machine is then settled by the field replay (`internal/boardengine/replay.go:210`) rather than by the owner path's rules. The diff also leaves the invariant's text false. Evidence: `doc.go:80-81`, `beamend.go:16-27`, `board.go:73-92`.
2. **LOW**, class design, `internal/battencli/cancel.go:578-586`. When EndBeam fails after RemoveRecords has committed the deletion of the records, the cancel returns before `PushRecords`. On the re-run, `RemoveRecords` makes no new commit, so `pushRecords` stays false and the cancel never pushes that records commit itself. It is left to whatever background push exists. The same gap already applies to a ClearClaim failure; the diff adds a second failure point after the commit. Evidence: `cancel.go:541-547`, `cancel.go:574-586`, `cancel.go:597`.

Outside this lens, noted only: draft members of the beam still make `refuseBeamCancel` refuse (`internal/battencli/beamland.go:119-133`, `internal/boardengine/parentbeam.go:32`). In the issue's scenario, a beam with members, the cancel never reaches EndBeam unless the members are moved off first.

Tool calls:
1. Read `orchcli.BeamDown` and the board's run-held and owner rules (`internal/orchcli/stop.go`, `internal/boardengine/runlock.go`, `internal/boardengine/doc.go`). My context lacked what BeamDown stops and whether the EndBeam write passes the run lock and the owner-writer invariant.
