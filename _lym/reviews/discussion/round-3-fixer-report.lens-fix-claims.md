```yaml
lens: fix-claims
outcome: reported
```

Task: check that every claim the fix added or rewrote about bounds, behaviour or code holds against the code at HEAD.

No findings.

Claims checked and confirmed:
- D2 says "Today that push sends the code side first." Confirmed: in `internal/fabricengine/pushanchored.go`, the `step` function in `PushPairAnchored` pushes `pushSideCode` before `pushSideRecords`.
- D2 says `lym loom commit-records`' push is among the pair-push callers. Confirmed: in `internal/loomcli/wiring.go`, the `Push` dependency of `commitStatusDepsGuardedBy` calls `fabricengine.PushPairAnchored` with `StatusPushLockWait`.
- D2 names `fabricengine.StatusPushLockWait` as the push lock wait. Confirmed: it exists, its doc calls it the bounded push-lock wait `commit-records` passes, and it is also the teardown lock's default.
- D4 says "today it marks the board `done` before the parent push." Confirmed: in `internal/landingshed/finalize.go`, `markTaskDone()` runs right after `parentHandle.Merge` succeeds and before `landed()` calls `pushParent`.
- D10 says the session id stays in `orchengine.State`. Confirmed: `internal/orchengine/state.go` has a `SessionID` field.
- The Constraints row says the resume check stays in `internal/shuttleengine/claudeengine`. Confirmed: `resume.go` is there.
- D9 cites `internal/shuttleengine/carryover.go`. Confirmed: the file exists.
- D3 says the cancel's `--remove` path removes the pair itself. Confirmed: `clearRunPhase` calls `removePairAndBranch` in `internal/battencli/cancel.go`.

Tool calls:
- One Bash call read `orchengine/state.go`, the claudeengine directory listing, `carryover.go` and the `PushPairAnchored` call in `wiring.go`. My context had none of these.
- One Bash call read the doc comment above that `wiring.go` call, to confirm it is the synchronous status-commit push that `commit-records` uses.
