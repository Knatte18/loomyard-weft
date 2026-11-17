```yaml
lens: fix-affected-set
outcome: reported
```

Task: lens fix-affected-set. I checked each set-premise finding's callers and sibling sites.

**Findings**

1. **LOW**, `internal/dobbyengine/doc.go:292-294`, the orphan bullet.
   - The doc still says a first sight "posts one notice … and records the time in the first-sight record Deps.OrphanRecord names".
   - After F2 the record is written only once the notice posted, and a failed notice leaves the orphan out of the record and notices it again at the next probe tick.
   - The `dueOrphans` doc was updated, but this package-doc sibling was not, so a reader would still take the first sight as recorded regardless of the notice.
   - F2 is a set-premise finding about first-sight sites, and this is the one the fix missed.

2. **LOW**, `internal/dobbyengine/doc.go:310`, the kept-cancel marking bullet.
   - It says `Deps.MarkKeptCancels` writes "its finished keep cancel intent" for a cancelled run with no intent whose pair stands here.
   - After F1 it also re-commits an intent this machine already wrote but never committed, and writes nothing while a merge is active.
   - The `MarkKeptCancels` doc was updated, but this sibling still describes only the no-intent case.
   - The leftover.go caller comment at line 194 is the same kind of sibling, but I did not read it, so I cannot say whether it is stale.

**Sets checked and covered**
- **F3 callers:** the only production caller of `RemoveHandedOverPair` is `internal/beamcli/dobbydeps.go:443`, which discards the result, so the new return type holds. The `td.RemovePair` call at `internal/pairteardown/handover.go:147` is a different path whose sessions the review did not name, so it is out of scope.
- **F1 callers:** `commitCancelRecord` is also called from `internal/battencli/cancelcmd.go:307`. Its behavior is unchanged, since the merge check was only extracted into `refuseWhileMergeActive`. `MarkKeptCancels` has one production caller, `internal/beamcli/dobbydeps.go:313`.
- **F11:** `PairLastCommit` has one production caller, now covered.
- **F12:** `RemoveOrphanPair` has one production caller, `internal/beamcli/dobbydeps.go:446`.
- **`pattern/PATTERN-pair-remote.md:59-60`** and **`internal/beamcli/doc.go:439-447`** describe the orphan teardown at a level the fix did not change, so they are still correct.

**Tool calls**
- `grep` over callers of `RemoveHandedOverPair`, `RemoveOrphanPair`, `commitCancelRecord` and `MarkKeptCancels`, and the first-sight, orphan and `PairLastCommit` sites: my context lacked the caller lists.
- `grep` over the docs and PATTERN files for first-sight, orphan and kept-cancel text: my context lacked the sibling prose.
