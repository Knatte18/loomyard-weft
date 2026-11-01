```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1 and F2 change set for consistency with its surroundings.

No findings.

- `decision.go:650-681` keeps every existing refusal message and gives an unreadable sampler record the same refusal as before for any caller other than the run's parent. When the parent resolves and matches the caller, the call proceeds. When the sampler read and the parent lookup both fail, the sampler-read refusal wins, which is consistent. The existing "sampler session check failure" row in `decision_test.go` still hits `refuseSamplerRead`: the caller is `hub:slug:sampler`, the parent name is empty, so the caller differs from the parent.
- `doc.go:41-42` describes the admission rule only. It does not mention the sampler read-failure refusal, so it needs no edit.
- The new `decision_test.go` success row and its `samplerErr` field are wired through `f.samplerErr` into the fake's `samplerStrand`, the same path as the existing refusal row.
- The `samplersession_test.go` rows agree with `samplersession.go`:
  - Dirty-wake row:
    - Wake 2 raises the flag, `judgeDone` stops `guid-1`, and `ensureSession` restarts with resume id `session-1`.
    - The mail goes to `strand-2`, the second strand the fake starts.
  - Damaged-record wake row:
    - The reason strings match `decisionRecordRestoreWayForward`, as the existing rows already assert.
    - Stuck ends the call, and `endSession` finds `guid-1` already stopped, so the stop list stays `["guid-1"]`.

I left one item unchecked: whether `decisionRecordRestoreWayForward` contains "git -C _lym checkout". The existing damaged-record row asserts the same string and passes, so it holds.

Tool calls (1 of 6):
1. Read `decision.go` lines 644-690 and grepped `doc.go` and the tests for sampler cross-references. My context held only the pre-edit `decision.go`, and I needed the edited region and the `doc.go` text.
