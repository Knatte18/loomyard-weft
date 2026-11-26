```yaml
lens: fix-proposal
outcome: reported
```

Task: check the reviewer's proposals the fixer carried out (F16, F17, F18) and look for concrete cases where a proposed change is wrong.

**1. MEDIUM: D5 can name a commit from before the landing**
- Changed text: D5, "the landing commit is the first commit on the parent branch's first-parent history, oldest first, that holds the task's work by fabric's holds-work test (`Fabric.HoldsWork`)". The archive sentence uses the same search.
- Summary: when the task's work was already on the parent before the landing, the search returns an older commit, so the row gets the wrong landing commit, time and message.
- The case: a run whose landing changes no code path. `Finalize.landedDiffEmpty` (`internal/landingshed/finalize.go:758-771`) logs this as a real outcome. The run's work is then landed by hand or by an adopter, and another run lands after it.
  - `Fabric.HoldsWork` (`internal/fabricengine/parenttips.go:271-295`) holds whenever merging the task into a base leaves the base tree unchanged. For such a task that is already true at the task's fork point and at every first-parent commit after it.
  - Searching oldest first therefore returns the fork point or the first commit after it, not the hand merge.
  - The same happens when the task's change reached the parent earlier, for example by a cherry-pick.
- Evidence that ancestry is the better test on this path: the up-to-date branch of the merge fires only when the task tip is an ancestor of the parent head (`internal/fabricengine/merge.go:216-221`).
  - So on the no-record up-to-date path, the first first-parent commit that has the task tip as an ancestor is exact.
  - `HoldsWork` is needed only on the archive path, where origin may hold a squash. There the search should at least start after the task's fork point.
- Note: the reviewer's fix said only "holds the task's work". Naming `Fabric.HoldsWork` as the test was the fixer's own wording.

**2. LOW: D6 rule 4 cannot extract into the current directory**
- Changed text: D6 rule 4, "it removes an existing empty `<dir>` with a remove that deletes only an empty directory, right before it renames the sibling onto `<dir>`". Also the new extract criterion, "an extract into an existing empty directory writes the run there".
- Summary: `--extract .` run from an empty current directory is accepted as an existing empty `<dir>`, but the remove-then-rename step cannot complete for it.
- Evidence:
  - On Linux, rmdir(2) of `.` fails with EINVAL.
  - The sibling "named after `<dir>`" is unclear when `<dir>` is `.`.
  - When `<dir>` is the shell's own directory given by its absolute path, the remove-and-rename leaves the shell in a deleted directory.
  - Rule 2 and the refusal criterion name no refusal for this case. One fix is to resolve `<dir>` to an absolute path and refuse the current directory, naming a way forward.

No findings on F16's proposal. Seeding the report files back and rewriting each `Report` field under the lock covers every reader the review names: `prreviewask.go:118-121`, `parentseat.go:83`, `parentreview.go:314-316` and `judgeparent.go:236`.

Tool calls:
- Read `Finalize.landedDiffEmpty` in `internal/landingshed/finalize.go:758-778`, to confirm that empty-diff landings happen in practice. My context held only its call site.
