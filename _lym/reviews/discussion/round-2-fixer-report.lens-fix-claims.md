```yaml
lens: fix-claims
outcome: reported
```

fix-claims: I checked every claim the fix added or rewrote against the code at this worktree's HEAD.

Claims checked:
- **D3, the LLM driver sentence (`DriverLLM` reads only the `hand_back` mark for re-launch and the binary watch; a blocked escalate goes to the arbiter first):** holds. `contracts/stencils/shed/shed-template-driver.md:60` sends a `blocked` stop to `## Arbiter` first. Lines 98, 101 and 145 and guide lines 67-68 exempt only `hand_back`. A fork re-launch applies only at error and interrupted stops (lines 74-88).
- **D3, batten's re-step scoped to batten with one driver hand-over re-step per D4's Bound:** holds, and it matches D4.
- **D5, the status spec bullet:** holds against `contracts/specs/loom-status-spec.md:94-97`.
- **Criteria, the re-check clause:** holds. `recheck` (`internal/sheddriver/drive.go:752`) builds a moved stop with `stopFromStatus`.
- **Criteria, the open-attempt clause:** holds. `resumeOpenAttempt` (`drive.go:637`) reuses an open arbiter attempt when an arbiter is wired.
- **Criteria, the fresh-start clause:** holds only in part; see the finding.

Findings:

- **LOW**, about the acceptance criterion clause "a driver starting at a halted escalate stop with no open attempt re-steps the row, and only a re-escalation reaches the arbiter":
  The claim leaves out the start check.
  A driver starting at any blocked stop first runs `checkBlockedStart` (`internal/sheddriver/drive.go:618-630`).
  Loom wires that check as `StartCheck: c.finalizeGuard()` (`internal/loomcli/drivewiring.go:301`), which is `finalizeDirtGuard(loomrecipe.IsFinalizeRow, c.parentCodeDirt)` (`internal/loomcli/finalizeguard.go:72-73`).
  So a halted escalate at Finalize with code dirt on the parent hands back to the orch with nothing stepped; it does not re-step.
  The clause should say "re-steps the row unless the start check refuses it, which hands back", or name that exception.

Tool calls beyond my context:
1. `sed` of `drive.go` lines 170-200: to see `run`'s start sequence, which my context showed only as grep hits.
2. `grep` of the driver stencil's lines 1-89: to confirm where a blocked stop goes and where the fork re-launch applies, since my context held only lines 90-150.
3. `sed` of `drive.go` lines 600-680 and 745-790: for the bodies of `checkBlockedStart`, `resumeOpenAttempt` and `recheck`.
4. `grep` for `StartCheck`: to find loom's start-check wiring.
5. `grep` of `finalizeGuard`: to see what loom's start check refuses.
