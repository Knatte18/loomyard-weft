---
dispositions:
  - id: F16
    disposition: fixed
  - id: F17
    disposition: fixed
  - id: F18
    disposition: fixed
---

# Round 2 fixer report

## Changes

All changes are in `_lym/discussion/decision-record.md`.

- **F16 (D3):** the seed now writes each tracked report copy back to the live report path, `ParentReviewFiles.ReportPath` of its ask number, and rewrites every seeded ask's `Report` field to that path, under the record's lock before any reader sees the record.
  The "So after an adopt or takeover" sentence now says readers read each done ask's report.
  Added the D3 criterion: after the live record and live reports are deleted, the PR-Review standing read and the round settle of a done ask succeed.
- **F17 (D5):** the opening no longer says the landing commit is the parent's head.
  The parent's head is used only when the merge makes a commit.
  When the merge is already up to date and no record exists, the landing commit is the oldest first-parent commit of the parent branch that has the task branch's tip as an ancestor, found through fabric.
  The archive of a held parent merge finds the commit the same way on the reset branch.
  The proposal's "holds the task's work" is made precise in two ways:
  - **Ancestry first.** The merge reports up to date only when the tip is an ancestor (`internal/fabricengine/merge.go:216-221`).
  - **`Fabric.HoldsWork` as a fallback.** It is used only for a squash landing on the archive path, and only for commits after the task branch's merge-base.
  The sentence saying the archive re-reads the head is gone.
  The rationale now says that a landing this Finalize did not make is found in the parent's history.
  Added the proposed criterion for a hand merge with another commit on top.
  The Notes for the plan writer now place the search beside `Fabric.HoldsWork`.
- **F18 (D6 rule 4):** the sibling is created with a create that fails when the path exists.
  An existing sibling, including a symlink, is refused with the way forward "remove the sibling by its path".
  An existing empty `<dir>` is removed by a remove that deletes only an empty directory, right before the rename, so the rename never targets an existing directory.
  A failed rename after that removal leaves `<dir>` absent.
  The refusal criterion gains the leftover sibling.
  The extract criterion gains an extract into an existing empty directory.

## Self-check

Self-check fixes, all in `_lym/discussion/decision-record.md`:

- **D3:** every report copy is written before the record copy, so a tracked record copy never names a done ask without its report copy. A criterion pins this.
- **D5:** the history search is bounded as described under F17 above. The hand-merge criterion also covers a task whose commits leave its landed diff empty.
- **D6 rule 2:** `<dir>` is resolved to an absolute path. The current working directory, or a directory that holds it, is refused with a way forward, and the refusal criterion names it.
- **D6 Bound:** the sentence now counts the removal of an empty `<dir>`.
- **D6 criteria:** the part-way-failure criterion now matches rule 4's absent case.

| Lens | Severity | Finding | Disposition |
| --- | --- | --- | --- |
| fix-consistency | MEDIUM | D6 part-way-failure criterion contradicts rule 4's absent `<dir>` | fixed (merged with fix-claims LOW) |
| fix-consistency | LOW | D6 bound sentence omits the removal of an empty `<dir>` | fixed |
| fix-claims | MEDIUM | D5 `HoldsWork` search, oldest first, can name a commit before the landing for an empty-diff task | fixed differently: ancestry test first, `HoldsWork` only after the merge-base, not the proposed "start after merge-base" alone (merged with fix-proposal MEDIUM) |
| fix-claims | LOW | D6 criterion versus rule 4's absent `<dir>` | fixed (duplicate of fix-consistency MEDIUM) |
| fix-way-forward | MEDIUM | D3 seed has no answer for a done ask whose report copy is missing after a kill | fixed as proposed: report copies are written before the record copy |
| fix-proposal | MEDIUM | D5 search names a pre-landing commit | fixed (duplicate of fix-claims MEDIUM) |
| fix-proposal | LOW | D6 cannot extract into the current directory | fixed as proposed: refuse the current directory or one that holds it |

Every fork returned a report.

## Disputed

None.

## Deferred

None.

## Rulings

None.
