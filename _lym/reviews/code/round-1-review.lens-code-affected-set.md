```yaml
lens: code-affected-set
outcome: reported
```

## Lens report: code-affected-set

**What I checked:** for each change in the diff, everything outside the touched lines that calls, reads, asserts or documents it.

**Per change, the set and whether the diff covers it**

1. **`DiscardTaskBranch` takes a new `rerun` parameter.** Callers are `battencli/cancelcmd.go:288`, `battencli/cleanup.go:187` and every test call site; all are updated, and the build gate decides the rest. `battencli/cancel.go:804` already names `lym batten cancel <slug> --remove` and is consistent. Covered.
2. **The `Rerun` field and its `rerun` JSON key in `remoteTeardownJob` and `PendingTeardown`.**
   - `childHold.finish` encodes the job to a child spawned from `selfexec.LymBinary`, which is the same binary. Covered.
   - A record has two kinds of reader:
     - `PendingTeardowns` and `pendingTeardownWayForward`, which now carry the field;
     - an older binary decoding through tolerant `encoding/json`, which drops the key and falls back to the described wording.
   - No `DisallowUnknownFields` reaches it, so PATTERN-tolerant-records holds. Covered.
   - Whether this is a record format another binary reads, for the summary's "two-way" call, is the handler's merge-danger check.
3. **The discard way forward's wording.**
   - Tests that pin the empty-`Rerun` wording still pass the empty case: `remoteteardown_test.go:47`, `pendingteardown_test.go:59,62`, `pendingteardown_integration_test.go:58`, `teardownlock_integration_test.go:341`.
   - `unknownKindWayForward` (`remoteteardown.go:200`) still describes the command. That is correct: the job is unread, so no command is known. Covered.
4. **"hub already exists" gains a way forward.** `warpbinding_clone_integration_test.go:467` asserts with `strings.Contains`, so it still holds. `fabricengine/doc.go:564` is updated. Covered.
5. **`RefuseHubWithoutBolt` in `runReconcile`.** It runs before any record is built, and a bare `output.Err` matches the existing "nothing mutated yet" comment above it. `fabriccli/doc.go` and `fabricengine/doc.go` are updated. `tools/sandbox/SANDBOX-FABRIC-SUITE.md` asserts no reconcile message. Covered.
6. **`resolveMergeSources` returns an error.** Both callers in `merge.go` (lines 190 and 409) handle it.
   - The `mergeguards.go` header and the `merge.go:382` comment ("best-effort Fetch") stay accurate.
   - The layout read changes no tracking ref, so the guard stage stays read-only. Covered.
7. **`FetchPairTips` and `ReturnToPreviousTransition` return a layout error.**
   - `battencli/codetipsettle.go` takes them through the `Fetch` dep (line 119, returned as "fetch the tips of") and the `GiveUp` dep (line 163, logged by `codeTipSettling.Fail`). Covered.
   - `battencli/catchup.go:222` (`fetchRunForCatchUp`) is **not covered**. See finding F1.
8. **The `fastForwardWeft` layout guard.** Its only caller is `FastForwardPair` (`parenttips.go:384`). Warning-only; covered.
9. **The `plain_archive_tags` key in cleanup's envelope.**
   - Readers: `fabriccli/doc.go` and the `fabric.go` cleanup help are updated. `cleanupremove_integration_test.go` reaches it through the reach report.
   - No stencil or spec reads the key. Covered.
   - Its help text spells `refs/lym/`. See finding F2.
10. **The help and doc of O3** (`remove --help`, `cleanup --help`, the `DropLocal` doc). The "tags stay local" claim survives nowhere else (grep). Covered.
11. **Commit messages.**
    - `Root-Cause:` is on 3cba70ea4 and no `Pattern-Departure:` or `Loosened-Check:` trailer is present. I raise nothing on the trailer under this lens.
    - The tests changed in the range loosen no assertion; the discard-rerun test tightens one.
12. **Declared PATTERN changes.** The directive declares none, so there is nothing to land.

**Findings**

- **F1 — MEDIUM, design.** Location: `internal/battencli/catchup.go:222-228` (`fetchRunForCatchUp`).
  - The diff makes `FetchPairTips` return the layout error, which names the refspecs to restore (`codetipsettle.go:177-179`).
  - Catch-up wraps any `FetchPairTips` error as `dobbyengine.ErrOriginUnreachable`.
  - `catchUpFailure` (`catchup.go:64-67`) then prints "origin is unreachable … run lym batten run <slug> again once origin answers", which drops the refspec way forward.
  - Before the diff, a records store with no fetch refspec fetched without error, so this misreport is new: an operator is told to wait for an origin that already answers.
  - The function's own doc ("A fetch that fails reads as an unreachable origin", `catchup.go:216`) is now wrong for this error.
  - Evidence: `catchup.go:216-228`, `catchup.go:60-67`, the diff's `codetipsettle.go:174-182`.
- **F2 — LOW, consistency.** Location: `internal/fabriccli/fabric.go:377` (the cleanup help: "commands that move it under refs/lym/").
  - PATTERN-single-repo-hub says "No other package reads the refspecs, spells `refs/lym/`, or branches on the layout" (`pattern/PATTERN-single-repo-hub.md:32`).
  - The base `fabric.go` had no `refs/lym` literal. The same file's clone flag help builds the prefix from `fabricengine.RecordsNamespaceRoot` (`fabric.go:107`), and the new help line should do the same.
  - I rate it LOW rather than BLOCKING because it is prose and reads no layout, and `hardencli/cli.go:144,168` already spells the prefix in help text. The handler decides whether it counts as a constraint violation.
  - Evidence: grep for `refs/lym` outside fabricengine; `git show` of the base `fabriccli/fabric.go` (no match).

**Tool calls**

1. Grep for the changed strings and keys across the repo: tests and docs that assert them, which my context lacked.
2. Read the `FetchPairTips` and `ReturnToPreviousTransition` callers in `battencli/codetipsettle.go` and `catchup.go`, plus `remoteteardown_test.go` and a docs grep: one hop beyond the manifest.
3. Grep for `ErrOriginUnreachable` handling, the settle deps' error handling and the sandbox fabric suite: how callers classify the new error.
4. Read `catchUpFailure` and `settleCodeTip`: what the operator sees for the new error.
5. Grep for `refs/lym` outside fabricengine, the PATTERN line and the base `fabric.go`: to judge the help literal against PATTERN-single-repo-hub.
