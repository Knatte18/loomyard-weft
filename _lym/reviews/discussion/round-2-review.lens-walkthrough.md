```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end on one concrete case and checked the end state against the decision's purpose.

**Walkthrough list**

- **D1, purpose missed.** Case: a resume at Run-Shed after `lym batten cancel --remove`-style deletions on origin. The local task branch survives and is adopted under `AdoptLocal`, and its origin tracking ref outlived the remote delete. `Add` succeeds and the pair push is spawned. The second reconcile then reads the task branch gone before the push lands (finding F1).
- **D2, purpose missed on its main path.** Case: a resume at Land after a removal. The not-live sentinel fires, the failed stop is written, and the operator runs the two `git branch` restores, the task branch at the parent. On the re-run D1 adopts the restored local task branch without a warp probe, so D8 drops no tracking ref, and the second reconcile reads the task branch gone (finding F1). Separately, "newest" archive tag has no ordering (finding F2).
- **D2, ErrRemoteLeftover path: met.** Case: the local weft diverges from origin. The named fetch, tag, push and leased delete leave origin without the branch. On the re-run the probe answers live, D8 drops the weft tracking ref, and the pair is adopted.
- **D2, records-worktree-stands path: met.** Case: only the task worktree was deleted by hand. `Add` refuses at `add.go:328-331`, then `lym fabric remove` finishes the pair (`remove.go:150`) and archives it, and the not-live restore follows.
- **D3: met.** Case: `lym fabric remove` at Land, then the warning names D2's way forward. The Finalize stop text is unchanged (`finalize_owner.go:160`).
- **D4: met.** Case: origin's weft tip diverges from the local tip. The push is refused non-fast-forward and only logged. The local tip is tagged, and origin's tip is fetched by SHA and tagged. `Add`'s cover rule (`remoteleftover.go:100-103`) then reads it as not live.
- **D5: met.** Case: a tackler meets "missing necessary objects". `ls-remote` shows the tip, and the ancestry check reads the commit present.
- **D6: met.** Case: a removal with origin lacking the local tip. D4 pushes, the lease is the local tip, and the delete succeeds. Case: the cleanup sweep with a diverged origin tip archives both tips and leases on the origin tip.
- **D7: met.** Case: an `Add` of a slug during `Remove`. It waits on the exclusive lock, then forks. The push is spawned after `Add` returns and takes the lock shared. `WriteFailedStop`'s run lock is not held during PreRun (`shedverbs/run.go:31`), so D2's write is not Busy against its own process.
- **D8, purpose partly met.** Case: the 2026-11-03 replay. A forked weft and warp with origin absent drop both tracking refs, so the Seed-Child reconcile judges neither. An adopted local warp branch is not covered (finding F1).

**Findings**

F1. MEDIUM, design. Location: D8 (scope of the drop), D1 step 4, D2's Land restore.
Summary: D8 drops a stale tracking ref only for a forked branch or an adopted local *weft* branch. When the local *warp* branch is adopted, `add` skips the warp probe (`add.go:389-394`, `if !warpAdopted`). D1's `AdoptLocal` re-adopt and D2's restored task branch (`git branch <task branch> <parent branch>`) are exactly that case. So a task branch tracking ref that outlived its origin delete survives `Add`. The second reconcile still judges the branch, because `originTargets` keeps any branch with a tracking ref (`originreconcile.go:673-681`). It reads the branch gone (`judgeBranch`, `originreconcile.go:539-541`) before the spawned pair push lands. That is the race the record's own 2026-11-03 constraint describes, now on the warp side.
Concretely, D2's Land acceptance criterion ("the re-run resumes Land") fails whenever the removal that deleted the task branch on origin left its tracking ref, which is the record's own evidence of how removals behave (decision-record.md:175). Fix: extend D8 to an adopted local warp branch whose origin copy is absent, which means `Add` probes origin for an adopted warp branch under the adopt-only option.
Evidence: `internal/fabricengine/add.go:389-394`, `internal/fabricengine/originreconcile.go:539-541` and `673-681`, `decision-record.md:152` and `175`.

F2. LOW, consistency. Location: D2, Land, not-live bullet.
Summary: "the newest of the slug's `archive/<slug>/*` tags" has no defined order. Archive tags are lightweight, named by SHA and made by `git tag <tag> <tip>` (`archive.go:120`), so they carry no tagger date. Successive runs of the slug share the archive namespace (decision-record.md:168). An earlier landed run's tag also holds a child status at `done`. The record should say what "newest" is read by, for example the tagged commit's committer date, and whether "holds `done`" filters the candidates or only describes them.
Evidence: `internal/fabricengine/archive.go:111-122`, `decision-record.md:68`.

Tool calls:
1. grep of `internal/shedverbs` for PreRun and run-lock acquisition. My context lacked whether the run lock is held around PreRun, which decides whether D2's `WriteFailedStop` would read Busy against its own process.
