# Card 2 — config-changes-three-way

**Edit:**
- `internal/fabricengine#ConfigChanges`
- `internal/fabricengine#ReadConfigChanges`
- `internal/fabricengine/configchanges_integration_test.go`
- `internal/landingshed#Finalize.configChangeNotice`
- `internal/landingshed/finalize_test.go`

**Prosa:**
- `internal/fabricengine/doc.go`
- `internal/landingshed/doc.go`

**Intent:** Report only the per-worktree config files the task changed away from the parent's current copy (D4).
`ConfigChanges` gains a `ParentTip` string field, the SHA of the parent's records-branch tip.
`ReadConfigChanges` keeps its merge-base as the fork point and its first `git diff --name-only <base> <tip>` over the scoped pathspec, and adds a second `git diff --name-only <parentTip> <tip>` over the same pathspec; `Files` is the sorted intersection of the two, so a file is reported only when the task's tip differs from both the fork point and the parent's tip.
A parent commit that reached the task through a records sync differs from the base but equals the parent's tip, so it drops out; a parent change the task never synced is unchanged on the task side, so it drops out; a task edit the parent later made identically equals the parent's tip, so it drops out.
The existing table in the integration test gains three rows: a parent-side change synced into the task yields no file; a task edit identical to the parent's current copy yields no file; a task edit that still differs from the parent is reported, and the result's `ParentTip` is the parent branch's SHA.
`Finalize.configChangeNotice` names the parent tip beside the base and tip it names today (`base %s, tip %s, parent tip %s`), and the `TestFinalize_ConfigChangeNotice` rows that read the notice add the parent tip to what the line must carry.
Both package docs describe the three-way comparison in the sentence that describes the diff today.

**ImpactSummary:** One read-only git reader gains a field and a second diff, one notice formatter prints it; the only caller is `internal/loomcli`'s `ConfigChanges` closure, whose signature is unchanged.
