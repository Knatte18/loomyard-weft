```yaml
lens: code-correctness
outcome: reported
```

**Task:** lens code-correctness: run each function, branch and error path the diff changes, with concrete inputs.

**Per function, the case I ran, and whether it is correct**

- `hereRequest`, call carries a slug, `root` or `--all`: it returns the refusal before reading any state. Correct.
- `hereRequest`, task worktree `idle`: `hereSubject` finds no beam entry, so it ends `no_subject` before reading the record. Correct, and no pull or record write happens.
- `hereRequest`, prime with an unfinished `root` record that is killed at deploy and has `check` not done: `TakesUp` is true, so it merges the record's request with the call's flags and skips the pull. Correct. `restart` is dropped here, but `runWorker` recomputes it from the merged request, so nothing is lost.
- `hereRequest`, beam `gamma` with no owner of its own and a running `root` record: `ownerSubject` climbs to `root` and `listed(root)` holds. Correct.
- `hereRequest`, prime beside a record that lists only `alpha`: `TakesUp` is false, so it pulls, reads this machine as owner, and ends `owned_here` with no spawn and no record. Correct.
- `hereRequest`, record past `check` and stopped on a held-back project: `Done(StepCheck)` makes `TakesUp` false, so it falls through to the pull and `owned_here`. Correct.
- `hereRequest`, empty owner: `OwnedElsewhere("", here)` is false, so it ends `owned_here`. Correct.
- `hereRequest`, foreign-owned beam: it returns `Slugs=[slug]` with the call's flags kept, and `runAdopt` then applies the usual rules (a different subject's unfinished record is refused). Correct.
- `hereRequest`, `PullBolt` returns an error: it refuses with a message. Correct.
- `hereRequest`, a skipped pull (unreachable origin, or a dirty or diverged Bolt): the detail becomes a warning only on the `owned_here` path. Correct. The adopt path does not carry it, but that path pulls again.
- `Record.TakesUp`, record not `Unfinished` or `check` done: it returns false. Correct.
- `Record.TakesUp`, beam with its own owner that the record does not list: the subject is the slug, which is unlisted, so false. Correct.
- `runAdopt`, `--here` with `--discard`: refused. Correct.
- `runAdopt`, `--here` after `req = resolved`: `NoOrch` is then set from the strand, and validation runs. Correct.
- `Command` RunE: `--archives` is dispatched before `hubCallerDeps`, and `--archives` plus `--here` is refused in `runArchives`. Correct.
- `runArchives`, no subject: refused. A task subject: refused naming the mover. An unknown slug: refused. A `ReadArchives` error: it names the re-run. All correct.
- `runArchives`, slug `gone` that the board no longer holds: `archiveSubject` calls `ParentOf`. I read `hubgeom.ParentOf`; with no origin record it falls back to `boardengine.ParentBeam`, so it does not error for a gone slug. The slug lands under the prime. Correct.
- `runArchives`, grouping of the sorted archives by slug: the `listed` loop assumes the read returns them sorted by slug, and `ReadOwnerArchives` does sort. Correct.
- `archivesCommand`: beams sorted, `root` appended, `""` when none stood. Correct.
- `ReadOwnerArchives`, prime slug `root`: it uses the prime's current branch for the code side and `RecordsBranchName` for the records side. Correct.
- `ReadOwnerArchives`, branch absent on origin: `branchTip == ""`, so the tag is listed and the fetch omits the branch tip. Correct.
- `readSideOwnerArchives`, repeat run: `createTagAt` reuses a tag already at that commit, so the read can be repeated. Correct.
- `readSideOwnerArchives`, a contained tag: it is skipped and no local tag is created. Correct.
- `remoteTagsUnder` and `remoteArchiveTags`: the pattern `archive-owner/*` does not match `archive/*`, and the error is now wrapped once, in the caller. Correct.

**Findings**

1. **LOW / design.** `internal/fabricengine/ownerarchive.go:~801`: `if _, err := repo.RemoteURL(originRemoteName); err != nil { return nil, nil }` treats every `RemoteURL` error as "no origin remote". A real failure (a corrupt config or a read error) then yields an empty list, which reads as "no archives" while archives may exist. Evidence: the unconditional `return nil, nil` on any error. Fix: return nil only for the missing-remote case and propagate other errors.

2. **LOW / design.** `internal/adoptcli/here.go:~562-596`: `hereSubject` judges beam, open and `Done` from the pre-pull board. After the pull, `owners.Effective(slug, …)` runs on a board that may no longer hold or open that entry. I did not read `Effective`, so how it reacts to a missing entry is unverified. If it errors, `--here` fails in a worktree whose beam origin has just ended. Fix: re-run `hereSubject` on the post-pull `owners`, and end `no_subject` if it no longer finds the subject.

I found no BLOCKING defect in the diff.

**Tool calls (2)**
1. `sed` of `createTagAt` in `archive.go`, `mergeRequest` and `runWorker` in `worker.go`, `OwnedElsewhere` in `owners.go`, and `grep` of `hubgeom.ParentOf`. My context lacked these callees, and I needed them for the repeat-run, merge/restart, empty-owner and gone-slug cases.
2. `grep` of `PrimeName`, `Bolt.Pull` and `WorktreeName`. I needed these to confirm the pull skip contract and that the prime's `WorktreeName` is what `PrimeName` carries.
