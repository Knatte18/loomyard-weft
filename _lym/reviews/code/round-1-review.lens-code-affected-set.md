```yaml
lens: code-affected-set
outcome: reported
```

My lens is code-affected-set: what each change forces outside the lines it touches, and whether that set is covered.

**Per change, the set I found:**

- **`existingHubError` whole-hub advice** (`clone.go:42`):
  - `warpbinding_clone_integration_test.go:468` now asserts the `mv` advice: covered.
  - `reconcile_missingbolt_integration_test.go:66` asserts the `--reset` advice for a hub with no Bolt checkout, which still holds: covered.
  - `fabricengine/doc.go:595-597` and `fabriccli/doc.go:16-17`: covered.
  - `SANDBOX-FABRIC-SUITE.md` F12 (lines 427-435) does not reflect the change: see F2.
- **`checkResetTarget` / `hubWorkOriginLacks` refusal:**
  - `resetHub` and the two early callers (`clone.go:282`, `clone.go:321`) are covered.
  - The `fabric.go` clone help (lines 64-66), `fabriccli/doc.go` and `fabricengine/doc.go:597` are covered.
  - The existing successful-reset tests (livestate `cloneHubResetRealHubCase`, `shortnamebinding_clone_integration_test.go:118/134`, `warpbinding_clone_integration_test.go:245`, `clone_singlerepo_integration_test.go:277/339`) keep a clean hub replaceable: covered.
  - `SANDBOX-CORE-SUITE.md:244` resets a scratch hub that holds no pair: unaffected.
- **`UpdateRemoteBranchLeased` now also wraps `ErrPushAuthRefused`:**
  - Its callers are `destroy.go:1670` (`remoteBranchRequest` update), `destroy.go:1785` and `1806` (seed pushes) and `destroy.go:1929` (`restoreSeededRecordsBranch`).
  - The classifiers that read `ErrPushAuthRefused` are `pushclassified.go:361/637` and `executor.go:176`.
  - `gitrepo/doc.go:49-50`, the `push.go:51` doc and the `fabricengine/doc.go:585-586` seed paragraph are updated: covered.
- **`seedPushOutcomeUnknown` / `seedPushCause`:** the only callers are `destroy.go:1786/1796/1807/1820`: covered.
- **`RunTeardownChild` keeps the record on an empty input:**
  - The `remoteteardown.go:465/504` doc and the `pendingteardown.go:1-5` file header agree.
  - `pendingteardown_integration_test.go` (row renamed) and `remoteteardown_test.go:572` cover it.
  - `docs/research/hardener-works-study.md:110-111` is a dated study of the old code and is no subject.
- **`recordsArchiveWayForward` condition removed:**
  - `pendingTeardownWayForward` (`pendingteardown.go:170`) and `failedRemoteSteps` (`remoteteardown.go:257`) are covered.
  - The tests at `remoteteardown_test.go:563-564` and `remoteteardown_rerun_integration_test.go:525` are updated.
  - `SANDBOX-FABRIC-SUITE.md:966` already words the way forward without the condition: covered.
- **`SettleGonePairTeardown` signature `(bool, error)`:**
  - Callers are `cancelcmd.go:310`, `fabric.go:1035` and the rerun integration test: covered.
  - The `fabricengine/doc.go:369-370` claim is false for cleanup-kind records: see F1.
- **`FetchRecords` added, `FetchPairTips` delegates to it:** callers are `catchup.go:235/237` and `battencli/codetipsettle.go:54`, and the doc and file header are updated: covered.
- **`deleteLeftoverUnderLock` tag push:** the `CleanupRemoteWarp` doc and `fabricengine/doc.go:377` are updated: covered.
- **`RemoveStepPendingTeardown` constant and the remove settle path:**
  - No test or doc enumerates the closed set of step values.
  - `fabriccli/doc.go:67-68` and `removeFields` (`steps` passes through) are covered.
- **Commit trailers:**
  - Every fix commit carries `Root-Cause:` naming the defect, not a symptom.
  - The `Loosened-Check:` trailer on `26d31303c` names the renamed row and a measured cause (a stat returning nil): holds.
  - No `Pattern-Departure:` trailer exists, and the directive declares no PATTERN changes, so nothing is owed there.
- **PATTERN fit:**
  - The raw `gitexec.Run` in `hubWorkOriginLacks` sits in fabricengine, which `PATTERN-fabric-boundary` and `PATTERN-gitrepo-client-boundary` allow.
  - `fabricengine/doc.go` documents only fabricengine.

**Findings:**

- **F1 — MEDIUM, consistency** — `internal/fabricengine/doc.go:370`
  - The new line "Each command a record names as its re-run calls it, so running that command finishes the teardown of a gone pair" is false.
  - Only `lym fabric remove --remote` (`fabriccli/fabric.go:1035`) and the cancel (`battencli/cancelcmd.go:310`) call `SettleGonePairTeardown`.
  - A cleanup-kind record (written at `cleanup.go:277`) names `lym fabric cleanup --apply --remote` as its re-run (`remoteteardown.go:200`), and the cleanup never settles a gone slug's record.
  - A discard record names its caller's command (`taskbranchdiscard.go:89`), which settles it only when that command is the cancel.
  - Fix: narrow the sentence to the removal and the cancel.
- **F2 — LOW, consistency** — `tools/sandbox/SANDBOX-FABRIC-SUITE.md:427-435` (F12)
  - The scenario still says "re-run `--reset` against a real hub and confirm the idempotent re-clone still works".
  - It has no step for the new refusal of a hub holding a pair or an unpushed code commit, nor for the new no-`--reset` advice for a whole hub.
  - The suite runs against a shared hub that may hold pairs, so the step as written can now meet a refusal it does not expect.

Outside my lens: the summary says the `--reset` refusal "comes before the clone touches the network". In the one-argument form it runs only in `resetHub`, after `probeWeftBinding` (`clone.go:327-374`).

Tool calls:
1. Grepped docs and tests for the changed messages and read the study's fault list: the manifest held no docs, fixtures or sandbox suites.
2. Read the `--reset` lines of the docs and sandbox suites, plus `reconcile_missingbolt_integration_test.go:50-80`: needed to check their wording against the new advice.
3. Read sandbox F12 and the pending-teardown scenario, and grepped `pattern/` for the changed symbols: rule and doc members of the set were not in the manifest.
4. Searched for enumerations of the remove step values: to check whether the new constant breaks a closed-set test.
5. Grepped the `fabric.go` help for the pair-not-found and re-run wording: to check the remove help text against the new settle path.
