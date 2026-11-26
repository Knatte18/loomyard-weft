```yaml
lens: fix-way-forward
outcome: reported
```

fix-way-forward: I checked whether every refusal, next step, row and test the fix added can be reached from the state it names and leads to progress.

**MEDIUM**: D5, "A resumed Finalize that finds the landing record reads the commit back from it and never re-reads the parent's head."
- **Summary:** when a parent push is left Stuck and the operator fixes it by hand, the landing record goes stale and nothing replaces it.
- **Evidence:**
  - When the parent push fails and no held merge is archived, Finalize goes Stuck. Its way forward is "push it by hand" or reconciling a diverged remote by hand (`internal/landingshed/finalize.go` `pushParent`).
  - If the operator resets the parent to reconcile it, the resumed Finalize's parent-side merge makes a new landing commit.
  - D5 then reads back the commit from the first attempt, so the index row and the D6 listing name a commit that is not on the parent.
  - The only replacement D5 names is the archive of a held parent merge.
  - The merge result already tells the two cases apart: `MergeResult.AlreadyUpToDate` (`internal/fabricengine/merge.go:111`).
  - So the record could be replaced whenever the merge commits, and read back only when the merge reports `AlreadyUpToDate`.

**LOW**: D6 Bound rule 4, "so a failure leaves `<dir>` absent", and the criterion "an extract that fails part-way leaves `<dir>` absent".
- **Summary:** for an empty `<dir>` that already existed, which D6 accepts, a failure cannot leave `<dir>` absent, so the test cannot be met for that input.
- **Evidence:**
  - D6 accepts "an empty or new directory".
  - A failure before the rename leaves an existing empty `<dir>` where it was, so it is not absent.
  - The rule and the criterion hold only for a `<dir>` that did not exist. They should say a failure leaves `<dir>` as it was.

**LOW**: D2, "a re-run, including the one the verb's failed-commit refusal asks for, writes nothing new".
- **Summary:** the re-run that the refusal asks for skips the copy, so the copy may never join the verb's commit.
- **Evidence:**
  - The refusal is given after the commit fails, so the copy is already on disk but uncommitted.
  - On the re-run the hash rule skips the copy. D2 says only that the verb "includes the copy in the commit", which reads as the copy it just wrote.
  - `pullRequestRecordCommitter` commits only the paths it is given: `CommitAnchoredPaths(..., []string{rel}, ...)` in `internal/loomcli/landingdeps.go`.
  - Unless the commit stages the PR findings subdirectory, the copy reaches origin only through a later status commit. That contradicts D2's criterion that the copy is in the commit `lym loom pr-findings` makes.

Outside my lens, one note for another lens: D1's tracked `.gitignore` is itself written into the kept directory. If it is written atomically, its own `.tmp-*` temp file exists before the ignore rule does.

Further tool calls:
- grep for `AlreadyUpToDate` in `internal/fabricengine`: my context did not show whether the merge result tells a no-op apart from a new commit.
