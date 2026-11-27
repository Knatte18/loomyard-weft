---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: "_lym/discussion/decision-record.md:163"
    summary: "D5's way forward for a refused landing push on the prime's code branch is `lym fabric push` (:163, :165), which in the prime pushes the records side only (internal/fabricengine/pushanchored.go, PushPairAnchored with the prime). The refused-push record of the code branch therefore never clears, the pusher never starts again (runpusher start refuses while it stands), and D3 (:96) and its criterion (:349-350) rest on the same verb. The diverged variant (:164-165) also tells the operator to merge origin's parent in; after a takeover the new owner re-runs Finalize from origin and lands the same run, so that merge lands the work a second time, where today's adopterFirstRepair (internal/landingshed/finalize_owner.go) drops the local squash. The old machine's catch-up also hands the landed run over and removes the pair D13 left standing, and D5 does not say what then happens to the landing entry that holds that pair's records commit. PATTERN-refusal-way-forward is broken, since D5 (:177) puts this way forward on a blocked halt."
    fix: "In D5, name for the prime's code branch a command that pushes it, either plain `git push` in the prime's worktree (the existing prime code way forward in failedpush.go pushStep; D5's settle rule then writes done and clears the record) or state that `lym fabric push` in the prime also pushes the newest tip a landing entry records. Keep `lym fabric push` for a beam parent. State the diverged case separately for when origin's parent already holds the run's work under another commit: the way forward drops the local landing commit (`git reset --keep origin/<parent>`, or a rebase that drops it when other local commits stand), and the landing entry goes with no done write. State that a hand-over of a run drops its landing entry and pair hold, with what that leaves on the local parent and its way forward. Name the verb per target in D3 (:96) and in the D3 and D5 owned-elsewhere criteria, and add a criterion that runs the named way forward for a prime parent and sees the record clear."
  - id: F2
    severity: BLOCKING
    class: design
    location: "_lym/discussion/decision-record.md:156"
    summary: "D5 has the parent target's pusher read only the parent's owner before the parent push and the done write (:156-160, :316-317). A beam project carries its own owner, and adopting a beam moves the beam alone, so a landed beam run adopted by another machine while its push is owed is still pushed and marked done here. Today handedOverStop reads the run's own owner before the parent push and again before the done write (internal/landingshed/finalize.go, pushParent); the move to the pusher drops that guard. PATTERN-owner-machine requires a Finalize past its merge to run only where no other machine owns the run's entry."
    fix: "In D5 (:156-163) and the PATTERN-owner-machine constraint (:316-317), state that before each parent push the pusher reads the owner of the parent and of every run whose landing entry the push carries, and before each done write that run's owner again. A run owned elsewhere is the same refusal as a parent owned elsewhere, naming `lym adopt` for that run, and the decision says whether it also holds back the other entries on that parent. Add a D5 criterion: a beam run landed on the prime and adopted by another machine while its push is owed gets no push and no done write."
  - id: F3
    severity: BLOCKING
    class: design
    location: "_lym/discussion/decision-record.md:148"
    summary: "D5 says origin never reads an ended entry whose parent lacks the run's work (:148), but D13 gates only Worktree-Teardown and the dobby's landed-pair teardown. `lym fabric remove` of the standing pair goes through the same composite, which reads the child's Finalize as done and marks the board entry done (internal/pairteardown/teardown.go RemovePair, claim.go settleBoardClaim). Its detached teardown child pushes the local records tip, held commit included, to origin, and with --remote deletes the task branch there behind a gate that reads the parent branch (internal/fabricengine/remove.go). The landed run's own terminal refusal names `lym fabric remove <slug>` as a step (internal/runfsm/act.go), and D13 leaves exactly such a pair standing. No bound is stated."
    fix: "In D5 or D13, state that while a run's landing entry stands every teardown of its pair refuses before any mutation, `lym fabric remove` with and without --force or --remote included, with a way forward naming the owed parent push (wait for it, or the refused record's step). If a forced removal is to stay possible, state instead that it writes no done, pushes neither the held commit nor anything after it, deletes no origin branch, and what becomes of the landing entry. Add a criterion: `lym fabric remove` of a pair with a standing landing entry leaves origin's board and records without done and the held commit, tier by the lowest that fits."
  - id: F4
    carries: F6
    severity: MEDIUM
    class: design
    location: "_lym/discussion/decision-record.md:245"
    summary: "The kill between the merge-in and the record write is still open. D9 writes the merge record before the after-the-merge-in clean check, which is still after the merge-in commit, and the removal on a resume takes its base from the record (:246-247). A pass killed after the merge commit and before the write leaves no record on a first merge-in, or the previous merge's on a later one. The resume's before-the-merge-in clean check then finds the leftovers with no usable base and halts as in issue #242, and a task head read on the resume is already the merge commit. No D9 criterion drives this kill."
    fix: "In D9, have the merge-in pass persist the pre-merge task head under the run's scratch directory before it merges (its own field or file, named through the existing scratch path), keep it until the removal has run, and have a resume take the base from it, never from an earlier merge's task_head or a head read after the merge. Add to the D9 red-first criterion a resume after a kill between the merge-in and the merge record write, on a first merge-in and on a later one."
prior:
  - id: F1
    status: resolved
  - id: F2
    status: resolved
  - id: F3
    status: resolved
  - id: F4
    status: resolved
    answer: "The rebuttal holds: the catch-up is run-scoped and no landing target triggers it, and a Bolt pull before each owner read meets the concern about a verdict read before an absence."
  - id: F5
    status: resolved
  - id: F6
    status: open
---

### [BLOCKING] The refused prime landing push names a way forward that cannot clear it

**Location:** `_lym/discussion/decision-record.md:163`

**Issue:** `lym fabric push` in the prime never pushes the prime's code branch, so the refusal is a dead end.
The diverged variant can land a re-run's work twice,
and the record is silent on the landing entry of a run the old machine hands over.

### [BLOCKING] The parent target's pusher drops the run's own owner check

**Location:** `_lym/discussion/decision-record.md:156`

**Issue:** Only the parent's owner is read, so a beam run another machine adopted is still pushed and marked done here.

### [BLOCKING] `lym fabric remove` breaks the records hold

**Location:** `_lym/discussion/decision-record.md:148`

**Issue:** The teardown composite marks the entry done and pushes the held records while origin's parent lacks the work.

### [MEDIUM] A kill between the merge-in and the record write still loses D9's base

**Location:** `_lym/discussion/decision-record.md:245`

**Issue:** The base is persisted after the merge, so that resume halts on the leftovers D9 is meant to remove.

## Focus verdicts

- Land row: settled by D5 (:167-170), Scope and its criterion; no defect while the parent is only ahead by owed tips.
- Two runs on one parent: the landing entries, release order, ledger entry and all-clear are stated;
  see the lock-free pass under Rejected.
- Records hold: the catch-up, record commit, Worktree-Teardown and dobby teardown are covered; F3 is the route left.
- Parent owner read: the verdict, the refusal and the ordering after an absence are stated; F1 and F2 are the gaps.
- Refused mark: the dobby tick drops entries and a landed `lym fabric push` clears a pair's record; F1 covers the prime.
- D9 base: first and later merge-ins are settled; the kill case is F4.

## Rejected

These are not raised as findings, since an incremental round admits only new `BLOCKING` findings on changed hunks.
Each was read from the code and the record, none was driven.

- Serious, on text this round did not change as far as the prior findings show:
  a pull-request landing checks for an owed parent push before it takes the parent lock (:175).
  A local landing can record its entry while that run waits for the lock,
  and the run then merges on GitHub while the local parent is ahead, which the fast-forward after it cannot take.
  Re-checking under the lock closes it.
- Serious, same limit: the record does not say what replaces Finalize's handling of a rejected parent push,
  namely the protected-branch way forward and the archive of a duplicate local merge (`pushParent`).
  Later local landings also stack onto a refused parent and report done (:125, :133-134).
- `parentMergeTip` takes origin's tip, not the local parent's, so while a push is owed every second landing loses
  each lock-free race and gates under the parent lock.
- Parent-Align runs the same clean checks around its own merge-in as Finalize, and D9 changes only Finalize,
  so the issue #242 halt can arrive one row earlier on a tapestry run.
- Cancel's records push is of the prime's records (`internal/battencli/cancelcmd.go`), not the pair's,
  so listing it among the pushes the hold binds (:145-146) has no effect.
- No acceptance criterion names D5's rule for readers of the batten record at Land or Worktree-Teardown (:149-151).
- The catch-up fetch failure under Open risks is unchanged text, and Scope states why that fetch stays.

## Focus departures

- "The directive never licenses reading a prior round's files before your review is saved."
  While looking for the changed hunks I printed the first bytes of `round-2-snapshot.json`, which hold only file paths and
  content hashes.
  No prior review, report or verdict was read.
