```yaml
lens: fix-affected-set
outcome: reported
```

Task: re-run the set-based searches for F5, F6/F9 and F4 and check the fix covers every member.

**Findings**

1. **LOW**, F9 (stale order text). `docs/overview.md:740` still says "Teardown first pushes an `archive/<slug>/<tip>` tag … to its remote, … then deletes that branch locally and on the remote". The fix reworded `pattern/PATTERN-fabric-git.md`, `remove.go` and `fabriccli/doc.go` but not this twin. The order is now local tag, local delete, then the child's push and origin delete.
2. **LOW**, F8/F9 (stale sibling doc). `internal/fabricengine/cleanup.go:101` documents `CleanupBranchEntry.ArchiveTag` as the tag "pushed to the weft origin before this branch was deleted". `cleanupOrphan` now tags locally and deletes locally, and the child pushes afterwards. When the hand-off happens, the field is the local tag only.
3. **MEDIUM**, F4 (exemption covers more than the parent had). `SetGateExemptAll` now runs unconditionally in `teardownChildCmd` (`teardownchild.go:73`). The only parent that exempts itself is the removal (`fabric.go:1103`, `remove.go:48`). `cleanupOrphan` (`cleanup.go:278`) and the discard (`taskbranchdiscard.go:36`) hand over to the same child but are not exempt. Their remote steps therefore moved from gated to exempt, not from exempt to exempt as the code comment and the `doc.go` paragraph claim. The discard has non-operator callers such as batten and shed reset. The comment says "an explicit removal, discard or cleanup … had already exempted", which is false for the last two.
4. **No finding**, F5 (end sites). I checked every `runproc.End`, `EndSparing`, `EndRole` and `EndDobbies` caller. `pairteardown` `cancel.go:156`, `skip.go:81` and `teardown.go:310` all go through `endOwnedProcesses`, which now spares `RoleTeardown`. `tendercli/deps.go:114` and `deps.go:333` use `runproc.End`. They are the deliberate operator end, and `teardownLockRefusal` names them. `anchordeps.go:243` spares only push and sync, but it ends dobby owners, so a teardown child is not in its set.
5. **No finding**, F1 (callers of the changed signature). `deleteTaskBranchOnOrigin` and `deleteTaskBranchAtTip` have no caller besides `runRemoteTeardown`, `deleteTaskBranchAtTip` itself and the updated `export_test.go`.

**Left unchecked**
- F2 and F3 coverage across job kinds. I did not check whether `RunTeardownChild` rows exist for the cleanup and discard kinds. The new integration test drives only `remove`.

**Calls made**
1. `grep` over process-end call sites, for F5; my context held only the `pairteardown` side.
2. `grep` for stale archive-order wording across `pattern`, `docs` and `internal`, for F9 and F8; my context held only the files the review named.
3. `grep` over every gate-exemption site, for F4; my context lacked which parents exempt themselves.
