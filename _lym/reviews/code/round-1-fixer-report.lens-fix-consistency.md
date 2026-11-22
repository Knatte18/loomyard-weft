Task: fix-consistency lens over the change set (F1–F5 commits and the summary edit).

**Findings**

1. **MEDIUM** — `_lym/landing/summary.md`, `## What changes`, first bullet. It still says `gitrepo.Fetch` "runs `git fetch --prune`, so every plain fetch drops the tracking ref of a branch origin deleted."
   - After F1 that is false: `Fetch` runs a plain `git fetch`, then `git fetch --prune --no-tags origin <refspecs>` over the `refs/remotes/` refspecs only, and never prunes a local tag.
   - The same file's intro says `gitrepo.Fetch` "did not prune", which is fine as history.
   - Evidence: the first bullet of `## What changes` in the file as read; `internal/gitrepo/pull.go:73`.

2. **MEDIUM** — same file, second bullet. It says the code-tip fetch covers "the pair's when it stands here and the prime's records otherwise."
   - F3 added a prime code-store fetch (`gitrepo.New(location.WorktreePath()).Fetch()`) to the `hubRunVerdict` closure in `internal/battencli/arm.go`.
   - The summary does not mention it.

3. **LOW** — same file, `## Tests`. Three entries are stale:
   - The `TestFetch_BranchDeletedOnRemote_DropsItsTrackingRef` entry omits the new `TestFetch_RecordsRefspecs_PrunesTrackingRefsButKeepsLocalTags` (F1).
   - The `TestJudgeAfterCodeTipFetch` entry omits the failed-read-after-fetch row (F4).
   - The `TestAct` entry mentions one new row; F5 adds a second, for the case with no tip and no machine.

4. **LOW** — `## Design decisions` records that the prune was chosen over dropping the ref in teardown, but says nothing about why the prune is limited to `refs/remotes/`. The reason is the records store's `+refs/lym/tags/*:refs/tags/*` refspec. This is optional and only affects a reader of the summary.

Every code and doc twin I checked agrees with the change:
- `doc.go`'s Fetch bullet, the `pull.go` file header and the `Fetch` doc comment all say "tags never pruned".
- The `hubRunVerdict` doc comment now says it fetches records and code refs.
- The `judgeAfterCodeTipFetch` doc comment is still accurate.
- `grep` finds no other `--prune` use or doc in the repo.

Calls made:
- One `grep` for `--prune` across the `.go` and `.md` files, plus a read of `summary.md` lines 1–46, because my context held only the Merge danger section of the summary.
