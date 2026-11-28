```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each change in the diff to a goal point of the board entry, and checked each goal point for a change that builds it.

**Decisions.** The answer key is the decision record alone, whose `## Goal` holds six faults to confirm with a test that fails first and fix. The record has no `Added after Discussion` entries.

- **G1:** a refused credential during a one-repository seed keeps the hub and says origin may still take the push.
- **G2:** an empty teardown job clears an earlier teardown's pending record.
- **G3:** the `cleanup` remote sweep counts local tags as landed and never pushes them.
- **G4:** `--reset` is advised without regard to unpushed work.
- **G5:** `fetchRunForCatchUp` reads no records layout.
- **G6:** a `lym fabric remove` pending record can outlive its pair.

**Per change, the decision it traces to:**

- **C1, seed auth refusal → G1.**
  - `internal/gitrepo/push.go:420-427` wraps `ErrPushAuthRefused` in `UpdateRemoteBranchLeased`, with doc updates at `push.go:51,398` and `gitrepo/doc.go:49-50`.
  - `internal/fabricengine/destroy.go:1846` (`seedPushOutcomeUnknown`) and `destroy.go:1949` (the `seedPushCause` case) change the seed's handling.
  - `fabricengine/doc.go:584-586` documents it, and the test `RefusedCredentialRemovesTheHubAndNamesTheCredentials` in `clone_singlerepo_integration_test.go` covers it.
- **C2, empty teardown job keeps the record → G2.**
  - `remoteteardown.go:493` drops the clear, and the `RunTeardownChild` doc changes at `remoteteardown.go:462-468`.
  - Tests: `pendingteardown_integration_test.go` (row renamed `NoRemoteStepsKeepsTheEarlierRecord`, with a `Loosened-Check:` trailer on 26d31303c) and the added record assertion in `remoteteardown_test.go:226-272`.
- **C3, sweep pushes local archive tags before delete → G3.**
  - `cleanupremotewarp.go:166-172` adds the push, with doc at `cleanupremotewarp.go:35-37,143-144` and `fabricengine/doc.go:253`.
  - Test: `PushesTheLocalArchiveTagThatHoldsTheCandidatesWork`.
- **C4, `--reset` advice and refusal → G4.**
  - In `clone.go`: `existingHubError` at 42-47, `checkResetTarget` and `hubWorkOriginLacks` at 815-880, and the `resetHub` dirtiness text at 798.
  - Docs: `fabricengine/doc.go:595-597`, `fabriccli/doc.go:16-17`, and the `fabriccli/fabric.go:64-68` help text.
  - Tests: `ResetOfAHubHoldingWorkOriginLacksIsRefusedAndKeepsTheHub`, plus the added move assertion in `warpbinding_clone_integration_test.go:468-475`.
  - The refusal inside `--reset` goes past fixing the advice alone. It still builds G4: it makes the `--reset` that `RefuseHubWithoutBolt` and `existingHubError` still name account for unpushed work.
- **C5, `FetchRecords` → G5.** `codetipsettle.go` adds `FetchRecords`, `FetchPairTips` reuses it, and the file header is updated. `battencli/catchup.go:237` is the call site, and a `FetchRecords` row is added to `codetipsettle_integration_test.go`.
- **C6, remove settles a gone pair's record → G6.**
  - `fabriccli/remove.go` changes `run` and adds `settleGonePair`. The `settle` wiring and `LoadConfig` change sit in `fabriccli/fabric.go:1009-1036`.
  - `fabriclayout/verbresults.go:491-493` adds `RemoveStepPendingTeardown`.
  - Docs: `fabriccli/doc.go:67-68` and `fabricengine/doc.go:245-246`.
  - Tests: four `TestRemover` rows.
- **Forced by C6:**
  - **C6a:** `SettleGonePairTeardown` now returns `(settled bool, err error)` (`pendingteardown.go:86-111`). The `cancelcmd.go:309-312` caller and the `remoteteardown_rerun_integration_test.go:98-114` assertions are adapted to it.
  - **C7:** the removal-only condition in `recordsArchiveWayForward` is removed (`remoteteardown.go:235-240`). It went stale once a gone pair's removal finishes its record. The expectations in `remoteteardown_test.go:67-68` and `remoteteardown_rerun_integration_test.go:59` are updated with it.
- **C8, 42c266c55 "Break three fabricengine comments at sentence boundaries" → forced** by the changed comments of C1–C6 meeting the comment lint. It carries no behaviour.

**Per decision, the changes that build it:**

| Goal point | Changes |
| --- | --- |
| G1 | C1 |
| G2 | C2 |
| G3 | C3 |
| G4 | C4 |
| G5 | C5 |
| G6 | C6, C6a, C7 |

No goal point is left unbuilt.

**Findings.** None. Every change traces to a goal point or is an edit a traced change forces, and every goal point has a change that builds it. No `Pattern-Departure:` trailer appears in the range, and none is needed.

Tool calls: none. The inherited context held the diff, the commit log and the decision record.
