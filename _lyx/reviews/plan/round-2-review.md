---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/35-beam-promote.md:38
    summary: "The root-pair refusals of cards 35 and 8 name the cause but no way forward, against the decision record's constraint that every new refusal names one."
    fix: "In card 35's Intent, end the pre-step-1 root-pair refusal with a trailing way forward clause that reuses the never-started refusal's route, for example: way forward: `lyx board upsert` a project entry under a slug of its own and run `lyx beam create` on it, since a root pair cannot become a beam. In card 8's Intent step 1, give the empty-ParentBranch refusal the same clause, and state in both cards that TestPromote and TestResetRecordsToParent assert the clause in the refusal text, not only the words root pair. Only Intent and test prose change, so no target or verify line moves and no validate-plan overlay is needed."
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/02-origin-beam-mark.md:24
    summary: "The missing-records-worktree branch of the missing-origin-record refusal, which names `lyx fabric reconcile`, has no named test row in card 2's TestMarkBeam or card 8's TestResetRecordsToParent."
    fix: "Add one row to TestMarkBeam in card 2 and one to TestResetRecordsToParent in card 8: a pair whose records worktree the test removes, so the origin record is absent because the worktree is gone, refused naming `lyx fabric reconcile` with no commit made. Name the row in each card's test paragraph beside the existing deleted-record row, which stays as the reach for the restore-from-history branch. Only test prose changes, so no target moves and no validate-plan overlay is needed."
---

### [MEDIUM] Root-pair refusals name no way forward

**Location:** `_lyx/plan/35-beam-promote.md:38` (pre-step-1 refusal), `_lyx/plan/08-weft-reset-executor.md:23` (step 1)

**Issue:** The decision record's constraints require that every new refusal names its way forward, and its acceptance criteria require each refusal to be reached with that way forward in its text.
Card 35 adds a refusal of a task whose pair's origin record has an empty `ParentBranch`, placed before step 1 so the run is not ended first.
That placement settles the ordering question: the refusal fires before the run is ended, marked or unclaimed, and `TestPromote` reaches it over a fake origin record with an empty parent branch and no step run.
The refusal's text, as the card words it, is "naming the pair a root pair, made by `lyx fabric add --adopt-local` over the default branch, that cannot become a beam".
That names the cause only.
No step lets the operator progress after it, while the card's other refusals each name one, and the never-started refusal beside it already names a route (a new project entry through `lyx board upsert` and `lyx beam create` on it) that serves a root pair too.
Card 8's step 1 refuses an empty `ParentBranch` as "a root pair with no parent to reset to" with the same gap, and its test plants that input.
`PATTERN-refusal-way-forward` binds only the webster, shed and loom packages, so the finding rests on the decision record's constraint and acceptance wording, and carries MEDIUM, as round 1's same-kind finding on card 5 did.
Round 1's fix for the root-pair refusal (its F10) is what this finding extends; the fixer report disputed nothing, so no dispute is re-raised.

### [MEDIUM] Missing-records-worktree branch has no reaching test

**Location:** `_lyx/plan/02-origin-beam-mark.md:24` (`TestMarkBeam`), `_lyx/plan/08-weft-reset-executor.md:30` (`TestResetRecordsToParent`)

**Issue:** Cards 2 and 8 word one refusal for a pair with no origin record that tells two causes apart: a missing records worktree names `lyx fabric reconcile`, and a record file absent from a present records worktree names restoring it from the records history.
The tests plant only the second input, a record the test deletes from a present records worktree, and that branch's way forward is correct: `git checkout HEAD -- <record path>` while the deletion is uncommitted, else `git checkout <commit>^ -- <record path>` from the commit `git log -1 --format=%H -- <record path>` names, then a re-run, and `reconcile` does not write an origin record into a present worktree.
The first branch is reachable (a pair with its task worktree present and its records worktree removed passes promote's pair-presence refusal) and no named test row reaches it, so its message is unasserted.

## Focus answers

- Card 35's `Promoted` check runs whether or not the status file is found, so a task whose promote stopped after step 2, with its claim and status file still standing, is refused naming `lyx beam promote <slug>`; the Intent says so and `TestStartGuard_Refuse` names a row for the found-status case.
  `armAt` reaches `refuseAtStart` for `run` and `step` before the seed and the claim, so the check is reached; `goto` only repositions a halted run and starts nothing.
  No finding.
- Card 24's Intent states what the on-demand session lets through: it holds the run's window and no orch strand, so `OrchLive` and the orch readers see no orch there.
  It states what ends it: the beam's pair teardown, which downs the pair's reed session before removing a worktree, with `lyx orch start` taking a live session as its own.
  No finding.
- A root pair refuses before step 1 (F1 covers the missing way forward).
- Cards 2 and 8 name a restore step that matches the input their tests plant, a record deleted from a present records worktree (F2 covers the other branch).

## Rejected

- A missing origin record read before step 1 of promote, so the run is not ended first: promote's step 2 refusal names the restore, a re-run repeats step 1 harmlessly, and a stopped promote already leaves the run ended by design, so the early end loses nothing.
- `goto` on a marked pair not being guarded: it moves a halted run onto a row and leaves it paused, so it restarts nothing.
