# Card 54 — refusal-spec-evidence

**Prosa:**
- `contracts/specs/refusal-spec.md`

**Intent:**
Bring the webster section of the refusal table in line with cards 47–53, which PR #321 round 3 asks for alongside the code.
Each edited row's Way forward cell quotes the clause its message now carries, as the file's own rule requires.
Keep semantic line breaks;
table cells stay on one line.

- The audit sentence above the table says a fork-contract write, a fork plan write, every fabric reference, and a parent write into the task worktree's tracked content, under the run's `_lyx` or webster's scratch directory, or into another worktree of the task repository are correctness, and every other finding is policy.
- Card 47: the two `audit: fabric-reference` rows become one `audit: fabric-reference` row: a fork or Master ran a command that references the fabric repo;
  correctness halt;
  `lyx webster recover-batch NN` at record-batch, and at run exit the pathless way forward below.
- Card 48: a new `audit: fork-plan-write` row: a fork's transcript wrote under the run's plan directory;
  correctness halt;
  `lyx webster recover-batch NN`, the batch failing with its report archived and the plan file named as a suspect path.
- Card 49: the `audit: parent-write, tracked or _lyx` row becomes `audit: parent-write, tracked, _lyx, scratch or another worktree`, its trigger naming all four places;
  the `audit: parent-write, elsewhere` row's trigger says a path outside every worktree of the task repository, or a git-ignored path in the task worktree outside `_lyx` and the scratch directory.
- Card 50:
  - `plan fingerprint mismatch`'s way forward names `lyx webster rebaseline --card NN` for each changed card, and the `--fresh` route when `00-overview.md` changed;
  - `plan drifted`, `zero execution batches` and `plan validation refused` name `lyx webster rebaseline --card NN` naming each edited card;
  - `rebaseline card set changed` keeps its trigger and way forward;
  - new `rebaseline unnamed card changed` row: a card file changed since the run recorded the plan and `--card` does not name it; correctness halt; re-run `lyx webster rebaseline` naming every changed card with `--card NN`, or restore the unnamed cards;
  - new `rebaseline overview changed` row: `00-overview.md` changed since the run recorded the plan; correctness halt; restore `00-overview.md`, or reset the branch to the run's start commit with git and run `lyx webster run --fresh`;
  - new `rebaseline bad card number` row: a `--card` value is not a positive integer; correctness halt; re-run naming card numbers;
  - `re-baseline persist failed, validate`'s way forward is re-running `lyx webster validate` alone.
- Card 51:
  - `run-exit audit correctness` and `pending audit findings` name restoring the named paths to the last batch head with git, then `lyx webster accept-audit`, and for a finding with no path, resetting the branch to the run's start commit and `lyx webster run --fresh`;
  - the `audit: parent-write` and `audit: fabric-reference` rows' run-exit clauses say the same;
  - new `accept-audit: suspect path differs` row: a pending finding's path differs from the last batch head; correctness halt; restore each with `git checkout <head> -- <path>`, deleting a path the head does not hold, then re-run `lyx webster accept-audit`;
  - new `accept-audit: finding not checkable` row: a pending finding has no path, or a path nothing the run recorded can check; correctness halt; reset the branch to the run's start commit with git and run `lyx webster run --fresh`.
- Card 52: new `fresh over a differing suspect path` row: `lyx webster run --fresh` finds a pending finding whose suspect path differs from the run's start commit; correctness halt; reset the branch to that commit with git, then re-run `lyx webster run --fresh`.
- Card 53: new `recovery suspect path not cleared` row: `recover-batch` finds a suspect path still holding the flagged content, or with changes the recovery's head does not hold; correctness halt; `lyx webster recover-batch NN`, the batch failing again with the path named for the next strand.
