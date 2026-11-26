---
dispositions:
  - id: F19
    disposition: fixed-differently
    reason: "Carried out (a) to (d) and the criteria, but the kept tip is not taken only before Finalize's first merge-in: the self-check showed the merge fixer commits on the task branch after a merge-in (internal/landingshed/finalize.go gateAndFix), so a tip kept once would miss the fixer's work on the squash path and in the no-net-change test. The kept tip is rewritten before each merge-in, except when the record holds one and the task's tip is an ancestor of the parent's head, so a fast-forward never moves it."
  - id: F20
    disposition: fixed
  - id: F21
    disposition: fixed
---

## Changes

All changes are in `_lym/discussion/decision-record.md`.

- **F19, fixed differently.**
  D5 drops the premise that an up-to-date merge means the tip is an ancestor of the parent's head.
  It now cites `squash: true` and `TestMergeCrucible_DerivedAlreadyUpToDateIsReadFromTheRecord` for the empty squash, and `Repo.MergeStart`'s `--ff` merge-in.
  - (a) The landing record keeps a task tip, and every search runs over that tip.
    The tip is rewritten before each merge-in, except when the record already holds a tip and the task's tip is an ancestor of the parent's head.
    So a fast-forward onto another run's commit never moves it, and a merge fixer's commit does.
    The review proposed a tip taken once, before the first merge-in. I did not make that change: the fixer commits after a merge-in, so a squash landing of the fixed work would match no commit, and (d) would report "no net change" for work only the fixer added.
  - (b) A pull-request merge Finalize makes on GitHub records the merge commit GitHub reports.
    The plan-writer notes say `pullrequest.Client.Merge` must return that SHA, and that `Finalize.mergeOnGitHub` records it.
  - (c) The up-to-date path with no usable record and the archive path share one no-record rule.
    It tries ancestry first, then a squash fallback that counts only a commit that holds the work by `Fabric.HoldsWork` while its first parent does not.
  - (d) When there is no net code change, or no commit qualifies, the landing record and the index row carry no landing commit and no landing time, and the row says which case applies.
    D5's field list now names that reason, and D6's listing prints it.
  - The acceptance criteria now cover:
    - the hand merge through a Finalize that does its own merge-in;
    - the pull-request squash, the hand squash and the adopter-first squash with a commit on top;
    - a task with no net change, whose row has no commit and says so.
- **F20.**
  D5 reads the landing commit back from the record only when fabric shows the commit is an ancestor of the parent's head.
  Otherwise it treats the record as absent and replaces it by the no-record rule.
  The sentence about a hand-reset parent now separates two cases: a reset to a branch that lacks the work gets a new merge, and a reset to one that holds it gets that branch's commit.
  A new criterion covers the protected-branch case: the push is refused, the work is merged on GitHub by hand, and the parent is reset to origin.
- **F21.**
  D1 replaces the tracked `.gitignore` with an ignore line in the records repository's `.git/info/exclude`, written only through `internal/fabricengine` as one more entry that `seedWeftArtifactExcludes` writes.
  That seeding runs at junction wiring, so a new pair has the line before its first write.
  It also runs before every records stage, through `ensureWeftLockDirAt`, so an already wired pair and an adopted pair have it before any stage.
  The line opens with `**/`, so it matches under any anchor.
  D1's rationale now cites `PATTERN-fabric-boundary`, and D3's reference now reads "D1's exclude line".
- Self-check changes:
  - D5's opening sentence now says the record holds the kept tip and the landing commit.
  - The plan-writer note names the GitHub merge-commit write site.
  - D1 pins the `**/` form of the exclude line.

## Self-check

- fix-consistency, MEDIUM: D5's opening sentence contradicted the new writes of the record. Fixed.
- fix-consistency, LOW: D5's field list did not show that the landing commit and time can be missing, and had no reason field. Fixed.
- fix-consistency, LOW: the notes did not name the GitHub merge-commit write site. Fixed.
- fix-claims: no findings.
- fix-proposal, MEDIUM: a tip kept once, before the first merge-in, misses the merge fixer's later commits. Fixed: the tip is rewritten before each merge-in, except after a fast-forward.
- fix-proposal, LOW: the exclude line's pattern form was not pinned, so it could miss a non-root anchor. Fixed with `**/`.
- fix-way-forward: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
