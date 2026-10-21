# Card 10 — failed-push-printing

**Create:**
- `plan:internal/fabricengine#FailedPushFields` -> `func FailedPushFields(hubPath string) ([]map[string]any, error)`
- `plan:internal/tendercli#TestStatus_PrintsOutstandingFailedPushes` -> `func TestStatus_PrintsOutstandingFailedPushes(t *testing.T)`
- `plan:internal/fabriccli_test#TestStatus_PrintsOutstandingFailedPushes` -> `func TestStatus_PrintsOutstandingFailedPushes(t *testing.T)`

**Edit:**
- `internal/fabriccli#addWeftVerbs`
- `internal/tendercli#statusCmd`
- `internal/cleanupcli#Command`
- `internal/cleanupcli#primeSegments`
- `internal/battencli#CleanupReport`
- `internal/battencli#Cleanup`
- `internal/battencli#battenCLI.cleanupCmd`
- `internal/cleanupcli#TestPrimeSegments`
- `internal/battencli#TestCleanup`

**Uses:**
- `internal/fabricengine#ListFailedPushes`
- `internal/fabricengine#FailedPushWayForward`

**Intent:** D10's visibility: `lyx fabric status`, `lyx tender status` and `lyx cleanup` print the outstanding failed-push records with the way forward for their class, and `lyx cleanup --apply`'s push result stops reading a spawned push as landed.
`FailedPushFields` renders `ListFailedPushes` as the envelope rows every verb prints: repo side, branch, class, last error line, first and last seen, count, `notified` and `way_forward` from `FailedPushWayForward`; an empty list is `[]`.
The `status` verb in `addWeftVerbs` adds `failed_pushes` beside `changes` and `merge_in_progress`, and its long help names the field; `tendercli#statusCmd` adds `failed_pushes` beside `status`; `cleanupcli#Command`'s envelope adds `failed_pushes` beside `applied` and `segments`, in the dry run and with `--apply`, and its long help names it.
`battencli#CleanupReport` gains `PushSpawned`, set when the records push was spawned, and `PushError` stays the spawn failure; `Cleanup` sets it, and `lyx batten cleanup`'s envelope and the `Output:` line of its long help, both built in `battenCLI.cleanupCmd`, and `cleanupcli#primeSegments`' batten segment print `push_spawned` beside `push_error`, so an empty `push_error` no longer stands for a landed push, and the outcome of that push is read from `failed_pushes`.

Tests: `TestPrimeSegments` and `TestCleanup` assert `push_spawned` for a pass that committed and spawned, and its absence for one that committed nothing; the two new status tests (tendercli's integration tier, fabriccli's external integration tier, each over a `hubforge` hub with a record written through `RecordPushFailure`) assert the `failed_pushes` row with its way forward, and `[]` with no record.
`lyx cleanup`'s printing is reached by card 20's verb test.

**ImpactSummary:** Additive envelope fields on three verbs and one report field; no field is removed, so JSON readers of the old keys keep working.

**Commit:** 10: print outstanding failed pushes in fabric status, tender status and cleanup
