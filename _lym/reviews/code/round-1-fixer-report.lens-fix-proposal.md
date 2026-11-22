```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals the fixer carried out as proposed (F1–F4, F6–F11) and report the cases where a proposal is wrong or breaks something.

No further findings on F1, F2, F6–F11 (the F5 change was the fixer's own and is outside this lens). Two on the proposals themselves:

1. **MEDIUM — F4's first-parent walk of the records branch.**
   - Changed text: `batten.go` `walk` (`LogQuery{FirstParent: true, PathPrefix: root, Since: b.floor}`) and `readCheckpoints` in `landings.go`.
   - Summary: a first-parent walk drops status commits that reach the records main line through a merge's second parent.
   - Evidence: `gitrepo.Log` with `PathPrefix` compares each commit only with its first parent (`LogQuery` doc). A merged-in pair commit therefore shows up only as the merge commit, so the intermediate status snapshots disappear from `touching[slug]`. The old walk was a full committer-time walk and saw them. `mergeguards.go` suggests records branches do get merged. I did not confirm that the records main line takes merges of pair commits.
   - Case: a pair branch with three status commits, merged into the records main line. The batten source then holds one snapshot instead of three, and `isFreshSeed` reads the merge commit's tree.
   - Why it matters: it changes what the report answers, which the F5 ruling forbids. The fixer's own note that records are conventionally linear (`AnchoredFileFirstParentHistory`) is the only support.
   - Reviewer's alternative: a `LogQuery` option that stops a full walk at the first commit before `Since`. That keeps the second-parent commits.

2. **MEDIUM — F3's bound on the seed stamp (`battenSeedMargin`, `reach`).**
   - Changed text: `snapshotsOfRun` (`reach = started.Add(-battenSeedMargin)`) and `widen(bound)`.
   - Summary: the bound loses a run's batten source when its pair was created more than a week before the loom seed stamp.
   - Evidence: the stamp is `shedrun.Seed.StartedAt`, "the time the run was seeded". The batten seed (worktree-create) is committed when the pair is created, and nothing ties the two within a week. Case: the pair is created at hour 0, the tapestry run is seeded at hour 250 (10 days later), and the tag is in the window. `reach` is hour 82, the walk stops at floor 82 and finds no seed, and `Sources[batten]` is false.
   - Before the fix, the walk widened to the whole history and found the seed. The change is a silent result change, and the new test row pins it as intended.

Tool calls:
- `grep` in `fabricengine/mergeguards.go` and across `internal`, for merges into the records branch. My context lacked any evidence on whether the records main line takes merges. The result was inconclusive.
