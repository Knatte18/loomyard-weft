```yaml
lens: fix-claims
outcome: reported
```

Task: check that the claims this fix added or rewrote hold against the code at HEAD (fix-claims lens).

**MEDIUM — D5, the new sentence "the landing commit is the first commit on the parent branch's first-parent history, oldest first, that holds the task's work by fabric's holds-work test (`Fabric.HoldsWork`)"**
For a task whose landed diff is empty, this rule names a commit from before the task ever landed.
- `Fabric.HoldsWork` (`internal/fabricengine/parenttips.go:271-295`) is true whenever merging the source into the base is clean and leaves the base's tree unchanged.
- A task branch that changes nothing against its fork point therefore "holds" at the fork point and at every later first-parent commit.
- Walking oldest first, the search returns the fork point or an earlier commit. It does not return the hand merge or the adopter's landing, and the row gets that commit's time and message.
- Finalize already handles empty landings: `landedDiffEmpty` is passed with the landing commit at `internal/landingshed/finalize.go:748` and defined at `:755-758`.
- The same problem hits the archive path, which the fix says finds the commit "the same way on the reset branch".
- The rule needs a lower bound so the search can't return a commit older than the task's work: for example, start after the task branch's merge-base with the parent, or for a true merge require the task tip to be an ancestor.

**LOW — D6 rule 4, the new clause "except a failed rename after that removal, which leaves `<dir>` absent"**
The acceptance criterion the fix left unchanged says otherwise: "an extract that fails part-way leaves `<dir>` as it was, absent or empty, and no sibling behind".
- Under the new rule 4, a `<dir>` that was empty can end up absent after a failure, so it is not "as it was".
- The criterion now contradicts the bound the fix states.

The other changed claims hold:
- **D3:** `ParentReviewFiles.ReportPath` and the ask's `Report` field exist (`internal/burlerengine/parentreview.go:157-159` and `:390-394`). Readers go through `state.ReadJSON` under `files.Lock` (`:197-198`), so "under the record's lock before any reader sees the seeded record" is achievable.
- **D5 archive path:** "resets the parent branch to origin's tip" matches the `fabricengine.ArchiveParentMerge` doc (`internal/fabricengine/handover.go:489-493`).
- **Plan-writer note:** `Fabric.HoldsWork` is in `parenttips.go`, as the note says.
- **D6 sibling creation:** a create that fails when the path exists (`os.Mkdir`) does fail on an existing symlink.

Further tool calls:
- One grep of `ReadParentAsks` and `landedDiffEmpty`. My context lacked whether readers take the record's lock, and whether Finalize handles empty-diff landings.
