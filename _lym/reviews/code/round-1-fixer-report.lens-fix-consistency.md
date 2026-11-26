```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of the fix's change set against its surroundings.

1. **MEDIUM**: `internal/websterengine/verifygate.go`, `NewVerifyGate` doc comment (F2). The step list still says "the first failure of any kind records HEAD as the pre-fix head". When the gate takes over an earlier run's `PreFixHead` through `earlierPreFix`, `preFix != ""`, so `fail()` records nothing.
   - The paragraph below it, which the fix rewrote, says the later run starts from the persisted head, so the two disagree.
   - The inner comment "fail records the pre-fix head on the first failure." has the same stale claim.
   - The step "once a pre-fix head is recorded" stays true.

2. **LOW**: `internal/websterengine/state.go`, `State.PreFixHead` comment (F2 twin, not updated). It says "the HEAD the latest verify-gate run started its fixes from. The gate records it at its first failed evaluation". After the fix, the latest run may start from an earlier run's head and record nothing. The sibling `FixedBase` comment, "the pre-fix head of the gate that passed", depends on this.

3. **LOW**: `contracts/stencils/webster/webster-body-verify-fix.md:20` (F2 reach, not updated). It tells the fixer that listed fix commits are ones "earlier fixers made above the pre-fix head: they did not clear the failure".
   - With the head kept across restarts, `commitsSince(preFix)` on a restarted run's first failure also lists commits from the earlier run.
   - Those can include ones that did clear an earlier failure, or a hand fix the parent committed under the stuck reason's "commit a fix on the task branch".

4. **LOW**: `internal/websterengine/runlevel.go:722` (F4 twin left behind). The comment still reads "Merriam may write nothing but its two contract files." F4 and F11 widened that statement in the stencil and `orch-role.md` to include the friction note.

5. **LOW**: `contracts/stencils/webster/webster-template-master.md:308` (F4/F12). Line 308 now says "your two contract files and the friction note" unconditionally. Lines 319 and 321 say "when one is named". With friction off, the prompt refers at line 308 to a note it never names.

6. **LOW**: `internal/websterengine/runlevel.go`, `pendingPathsWayForward` doc comment (F1 reach). It says "A cleared contract file and a path on the run's records side are exempt" from the reset route. With an empty or unknown `MasterRecordsBase`, `recordsSideStatus` now returns `onRecords=false`, so a records-side path does take the reset route (the new `NoBase` test row). The `AcceptPendingAudit` and `doc.go` records-side sentences don't state the no-base case either.

Further tool calls:
- grep for pre-fix and fixed-head prose across `internal`, `contracts` and the docs: my context held no search of those twins outside the files the fixer read.
- grep for "two contract files", `accept-audit` and records-side wording, plus `Git` implementers: my context did not show whether other statements or implementers of the changed rules existed. No other `Git` implementer than `realGit` and `fakeGit` turned up.
