---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: decision
    location: internal/loomcli/checklist.go:121
    summary: The checklist state line carries no waiting note, so the parent lock wait the parent added to the Waiting hook is no longer shown by the status strand
    fix: Decide and record it in internal/loomcli/doc.go. Either accept that the status strand shows only the state line and the rows and that the wait is read through lyx loom status and the driver pane's wait mark, and say so in the checklist verb's Long text. Or have checklistPoll call reviewWaitingFor(location) while the state is running and append the note to the state line the way shedverbs.statusWatchLine does, which also restores the gate, verify, shuttle and parent-review notes the old strand showed.
---

### [LOW] The checklist drops the Waiting note, including the parent lock wait this merge added

**Location:** `internal/loomcli/checklist.go:121`, with `internal/loomcli/reviewstatus.go:156-194`

**Issue:** The task replaced the `loom-status` strand's pane command, `lyx loom status --watch`, with `lyx loom checklist`.
The old command appended ` | waiting <note>` to the state line from `shedverbs.Hooks.Waiting`, which loom fills with `reviewWaitingFor`.
The parent's side of this merge extended that hook with `parentLockWaiting`, reading `waiting for the parent lock on <branch> held by <slug> <elapsed>`, and `PATTERN-wait-mark-display-only` now names the parent lock wait among the displayed waits.
`renderChecklist` builds its state line from `shedverbs.RenderStatusLine` alone and never calls the hook.
After the merge, the only consumers of the new parent lock note are one-shot `lyx loom status` and the wait mark on the driver pane.
A run blocked on the parent lock shows in the checklist as a running Merge-In row whose time keeps counting, with nothing saying it waits.
Decision record D7 prescribes exactly the state line, so the checklist matches the fasit; the parent's extension assumed a consumer the task removed, which is why this is filed as a decision for the author and not as a defect.
The rest of the interaction holds, as the checks below show.

## Checks that found nothing

- Every config load the task added in `orchcli`, `battencli` and `loomcli` (`reedengine.LoadConfig` in `overview`, `checklist` and `openWindowInPrimeSession`) passes `fabricengine.ConfigDirs(location)`. No `AnchorPath` form of a layered load remains in those packages, so `PATTERN-config-layers` is followed.
- `statusbuild.go` keeps only `statusSidecarPath`. The only caller is `sharedbootstrap.go`, which reads and writes the sidecar through `statuspane`, so no caller or test is left on the removed helpers.
- `checklistVisitsOf` counts the parent's new `reroute` outcome as a visit of the rerouting row, and only `goto` is excluded. Rows come from the recipe, so the Merge-In and Merge-Review rows show with no change.
- `rowNoticeDeps` and `InnerRunDeps` carry the task's `NoticeLog` beside the parent's `VerifyRows` and `ParentLock`. The parent's stale-base resume report goes through `p.report`, not a notice, so it is correctly outside the notice log.
- `battencli`'s `resolvePersistentPreRun` skips both `overview` and the lock group. `envkit.FullEnv` carries the task's `nilLegal` entries beside the parent's new fields.
- The merged `cmd/lyx/daemonwakeups_test.go` lists both `internal/parentlock/acquire.go` and `internal/statuspane/pane.go`, and `PATTERN-daemon-wakeups` names both loops. The task added no other unbounded wait.
- The merged budget rows for `battencli`, `battenshed`, `orchcli` and `loomrecipe` equal the top-level tests, without `TestMain`, that each package holds.
- The wait mark stays display-only on the task's side: the Selvage marker is a display-only pane marker that no Go decision reads.
