# Card 8 — layout-push-types

**Move:**
- `internal/fabricengine#PushFailureClass`
- `internal/fabricengine#PushFailureNotFastForward`
- `internal/fabricengine#PushFailureTransport`
- `internal/fabricengine#PushFailureAuth`
- `internal/fabricengine#PushFailureGone`
- `internal/fabricengine#PushFailureOther`
- `internal/fabricengine#PushFailureDeferred`
- `internal/fabricengine#PushFailureTeardownLock`
- `internal/fabricengine#PushFailureRewound`
- `internal/fabricengine#PushFailureAnchorChange`
- `internal/fabricengine#RepoCode`
- `internal/fabricengine#RepoRecords`
- `internal/fabricengine#RepoBolt`
- `internal/fabricengine#FailedPush`
- `internal/fabricengine#PushNotice`
- `internal/fabricengine#StatusPushLockWait`
- `internal/fabricengine#LockWaitUnbounded`
- `internal/fabricengine#MachineNames`
- `internal/fabricengine#MachineNames.Holds`
- `internal/fabricengine#PairPush`
- `internal/fabricengine#PushSpawn`
- `internal/fabricengine#SpawnResult`
- `internal/fabricengine#PushResult`

**Create:**
- `plan:internal/fabriclayout#FailedPush.PairSlug` -> `func (f FailedPush) PairSlug() string`
- `plan:internal/fabriclayout#MachineNames.IsZero` -> `func (n MachineNames) IsZero() bool`

**Delete:**
- `internal/fabricengine#FailedPush.pairSlug`
- `internal/fabricengine#MachineNames.isZero`

**Edit:**
- `internal/fabricengine#recordPushFailure`
- `internal/fabricengine#retryWeftPush`
- `internal/fabricengine#takeoverStop`
- `cmd/lym#destructiveGuardMutatingResultTypes`
- `internal/fabricengine/doc.go`

**Intent:** Move the push path's plain data into `internal/fabriclayout`'s `pushtypes.go` (D1, D7), and retarget every caller:
the failed-push classes and record, the push notice, the repository-side names, the lock-wait constants, the pair push and machine-name options, the push spawn request and result and `PushResult`.
Each is plain data; `SyncOptions`, `PushNotifier` and `PushFailedError` stay in the engine, since the first two carry function values and the error's message is built by the engine's way-forward helpers.

`RepoCode`, `RepoRecords` and `RepoBolt` move as the literals `code`, `records` and `bolt`.
The engine's private `failedPushSideCode`, `failedPushSideRecords` and `failedPushSideBolt` keep their literals for this card, so no engine name is an alias of a moved one; card 9 replaces them with the layout names.

Two unexported methods have engine callers, so they become exported layout methods with today's bodies:
- `FailedPush.PairSlug`, reading `RepoRecords` and `RepoCode`; `recordPushFailure` calls it.
- `MachineNames.IsZero`; `retryWeftPush` and `takeoverStop` call it.
`MachineNames.Holds` moves with its type from `takeover.go`.

`cmd/lym#destructiveGuardMutatingResultTypes`' `PushResult` row names `internal/fabriclayout/pushtypes.go`.
`doc.go`'s commit and push section names `fabriclayout.PairPush` where it names `PairPush.LockWait`; its rules are unchanged.
Every reference to a moved name, in the engine and in every other production and test file, becomes `fabriclayout.<Name>`, imports following as card 3 says.
The failed-push and push tests stay in the engine: each asserts a record the engine writes or a push it runs.
No behavior changes (D5): every record, class string and flag is the same as before.

**ImpactSummary:** Twenty-three push names move with mechanical retargets across the push callers outside fabric; three engine callers of two exported-in-place methods and one guard row change.

**Commit:** `8: layout-push-types`
