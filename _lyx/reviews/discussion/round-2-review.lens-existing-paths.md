```yaml
lens: existing-paths
outcome: reported
```

Task: existing-paths lens over the decision record. I traced each path that writes or reads the state a decision changes.

**Paths traced**

- **D9 hold (beam entry run status at Beam-Await, read as no hold):**
  - `checkProjectField` survives (stated).
  - `ComputeLayers` survives (stated).
  - `checkRunLocks`/`scopeChanged` **breaks**: the lift frees every scope field, see F1.
  - `rename` survives: a beam is refused by its own rule.
  - `beamForkPrecondition` survives (stated).
  - `ResolveBattenPrime` landing read survives (stated).
  - The orchcli refusal is removed, and D3 replaces it.
  - `loom peers` survives (stated).
  - `cancel.go` otherLeft survives: it still counts as left.
  - `cancelcmd` status clear survives.
  - `pairteardown.settleClaim` survives: it is only read at teardown.
  - `boardcli` run_settings producer check survives: the message is still accurate.
- **D9 status writes:**
  - loomcli `newCommitStatusSeam` (`lastBoardStatus` dedupe, warn-only) survives: Beam-Await's direct writes bypass it, and the next transition writes Parent-Align.
  - `shedengine` `CommitStatus` fires only after a transition, so re-entry does not rewrite Parent-Align before the row's Call. The interrupted-landing read survives.
- **D9 accept (dobby stop, then hold write):**
  - `ensureParentDobby` from a member's hand pass **breaks**, see F2.
  - `EnsureLive` from `lyx beam start` **breaks**, see F2.
  - Accept re-entry after a crash or a failed status write **breaks**, see F3.
- **D9/D11 max_runs:** dobby `slotsTaken` reads `RunView` (batten's own producer, Run-Shed). The record names the board status producer half as the source, so it survives as specified. The plan has to wire the read.
- **D5 RunHeld removal:**
  - `Effective` survives.
  - `tendercli/deps.go:177` survives (named).
  - `adoptcli/adopt.go:254,370` survives (named).
  - `battencli/owner.go:254` `resumeOwnerClaim` survives (named).
  - `beamcli/promote.go` is removed with D13.
  - The `owners.go` `owner_machine` payload check survives the drop-on-load.
- **D1 EnsureLive callers:** `lyx beam start`, adopt `StartDobbies` and the anchor move survive (named). `ensureParentDobby` names `lyx adopt --prime`, which D4's rewording covers.
- **D8/D10 recipe:**
  - `battencli/wire.go:499` project→beam recipe survives under D8.
  - Seed reading of `beam` survives under D8.
  - The `project.go` beam-flag message naming `lyx beam promote` survives under D13 and D4.
- **D11 Worktree-Create:** the project-absent-pair precondition (`wire.go:887-892`) is replaced by D11. `beamForkParent` runs before `top.Add`, and D9 adds the post-Add re-read. It survives as specified.

**Findings**

- **F1 — MEDIUM, design, D9 (hold bullet, `decision-record.md:154`).**
  - Summary: "the run lock leaves its fields editable" lifts `checkRunLocks` wholesale for a live beam entry. A board write can then change the beam's `project` (its parent), `kind` or `recipe`, or remove or merge the entry, while its pair, dobby and pawl run exist. The record bounds none of these.
  - Only the beam flag (`project.go` beam check) and the slug (`rename.go:47`) stay guarded on their own.
  - Evidence: `internal/boardengine/runlock.go` `checkRunLocks` (refuses absent-after or `scopeChanged`); `scopeChanged` compares Project, Kind, Recipe and Beam; `internal/boardengine/rename.go:47-49`.
- **F2 — MEDIUM, design, D9 accept (`decision-record.md:141`, with D1 `:38`).**
  - Summary: the accept stops the beam's dobby before it writes the Parent-Align hold. In that window, a member's hand `lyx batten run` (`ensureParentDobby`) or a `lyx beam start` (`EnsureLive`) reads no hold and restarts the dobby.
  - Nothing stops the dobby again before Beam-Down. A live dobby then drives the beam through Parent-Align to Finalize, starting released members that are each refused at Worktree-Create, with refused-start backoff churn.
  - Evidence: `internal/battencli/cli.go:506-540` (`ensureParentDobby` spawns when not live); D1 `:38` gates only on "holds the landing hold".
- **F3 — LOW, design, D9 accept retry (`decision-record.md:141-145`).**
  - Summary: the next wake retries an accept after a failed hold write, or re-enters after a crash between the dobby stop and the status write. That retry finds the dobby already stopped, so it did not stop it itself.
  - On a failing re-check, "restarts the beam's dobby only when this accept stopped it" then leaves the dobby down. The run sits back at Beam-Await with no dobby to start members, and no notice names `lyx beam start`.
  - Evidence: D9 lines 141, 144, 145.

**Tool calls**

1. grep of `CommitStatus(` in `internal/shedengine`: the context lacked when shedengine fires the status seam, which matters for the re-entry read.
2. `sed` of `internal/shedengine/run.go:880-910`: to confirm that `CommitStatus` fires only after a transition, with the next producer.
