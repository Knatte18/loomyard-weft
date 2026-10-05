---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md D4 and Acceptance criteria
    summary: D4 seeds the hub board.yaml but names no commit step, lock or Bolt route, while Acceptance says the seed is committed in _board
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D6
    summary: D6 --apply deletes a per-worktree board.yaml with no check that its content is already in the hub file, and the Bound limits path and mode but not content loss
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md D5
    summary: D5 states neither what happens when the _board commit or push fails after the file was written nor whether the edited hub file is validated before it is committed
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Open risks (first bullet)
    summary: The seed-source report in reconcile's result is a requirement living only under Open risks, so Plan-Write has no Decision or Acceptance criterion to build it from
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D4 and Auto-mode assumptions
    summary: D4 silently falls back to the template when no prime resolves, the same silent vocabulary loss D3 and the support log's rejected LoadOrTemplate alternative exist to avoid, and the prime premise is unverified
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Auto-mode assumptions (third bullet)
    summary: The boardengine/doc.go paragraph rewrite is plan-level detail, not an assumption, and the Constraints docs line already requires that file be updated
---

### [MEDIUM] D4 gives the seed no route into git

**Location:** `decision-record.md` D4, Acceptance criteria (fourth bullet).
**Issue:** D4 says reconcile seeds `board.yaml` in the board dir "where it runs today".
Today `ReconcileFabricAt` only writes the file (`internal/configsync/configsync.go`); neither `lyx fabric clone` nor `lyx fabric reconcile` commits it at that point.
Acceptance requires the seed to be "committed in `_board`".
Only D5 names a commit route (`fabricengine.Bolt` under the board write lock), and only for edits.
The plan writer would have to invent who commits the seed, under which lock, and whether it is pushed.
An uncommitted seed has the failure D5's rationale names: swept into an unrelated "board sync" commit, or lost on re-clone.
**Fix:** state in D4 that the seeded or reconciled hub-wide files are committed (and pushed) in `_board` through `fabricengine.Bolt` under the board write lock, the same route as D5, bounded to the hub-wide config files the reconcile wrote.
Say whether a push failure fails the reconcile or is reported and left for the next board sync.

### [MEDIUM] D6 can silently drop an unmerged vocabulary edit

**Location:** `decision-record.md` D6.
**Issue:** `--apply` deletes `_lyx/config/<module>.yaml` once the hub file exists.
The Bound limits what is deleted (path, registry mark, `--apply`, hub copy present) but not whether the copy's content is already in the hub file.
A pair that added a `feature` type or a label to its own `board.yaml` before this change, and whose hub file was seeded from the prime without it, loses that edit with no signal.
This is the attack-surface question's "guard that is bypassable": the per-worktree copy was the only record of the edit.
**Fix:** extend D6 so reconcile reports the keys of the per-worktree copy that the hub file lacks (the result already carries `Added`-style fields), and deletes the copy only when it adds nothing to the hub file; otherwise it reports the copy as divergent and leaves it.
Add the matching clause to the Bound and a divergent-copy case to the `lyx config reconcile` acceptance bullet.

### [MEDIUM] D5 leaves the failed-commit state and the pre-commit guard open

**Location:** `decision-record.md` D5.
**Issue:** Two gaps.
First, D5 commits "under the board's write lock as board sync does", but says nothing about whether the file write itself happens under that lock.
A write outside the lock can be swept into a concurrent `board sync` commit half-edited.
Second, it does not say what a failed `Bolt` commit or push leaves behind: an edited file on disk, nothing committed, and an error that must say so.
The strict load used to be the guard against a malformed `board.yaml` in one worktree reaching only that worktree; with a hub file, one bad edit made through `lyx config board` is committed to `_board` and breaks `lyx board` in every pair.
The Bound addresses who may edit, not what bounds a bad edit.
**Fix:** state that the edit is validated through the strict load before it is committed, and refused with the file left unchanged if invalid.
State that the write and commit happen under the board write lock.
State the failure outcome: file stays written, the error names the file and that it is uncommitted, and the next board sync commits it (or say it is reverted).

### [LOW] The seed-source report is a requirement, not a risk

**Location:** `decision-record.md` Open risks, first bullet.
**Issue:** "the reconcile result reports which source seeded it" is a behavior the plan writer must build, and `Plan-Write` reads the Decisions and Acceptance criteria, not risks.
By the completeness-before-leanness test it needs a home in a Decision or Acceptance criterion.
The post-deploy `lyx fabric reconcile` bullet, by contrast, is correctly a risk: it is an operational step for the PR description, with no code to build.
**Fix:** add to D4 that the reconcile result names the seed source (prime copy, template, or existing file), and add it to the fourth Acceptance bullet.
Trim the Open risks bullet to the residual risk once F5 is settled.

### [MEDIUM] D4's template fallback contradicts D3's reasoning, and the prime premise is unverified

**Location:** `decision-record.md` D4, Auto-mode assumptions (second bullet).
**Issue:** The support log rejects `LoadOrTemplate` because a silent degrade to the template "drops the hub's custom types and labels, the same symptom as the bug".
D4 does the same degrade at seed time: with no resolvable prime, or a prime without a `board.yaml`, the hub file becomes the template, silently from the operator's point of view (only reported afterwards, per the Open risk).
That is the worst case exactly where a hub has live vocabulary but no checked-out prime.
The premise "the prime is the worktree on the warp default branch, found by worktree listing" is stated as an assumption yet carries D4's whole seed source.
It is unverified for a hub whose default branch is not checked out anywhere, and the record does not say what the seed does then.
Separately, a prime's per-worktree copy can itself be behind an unmerged pair's edits, which the Bound does not mention.
**Fix:** decide the no-prime case: either refuse the seed with a way forward naming the hub file path to create or the prime to check out, or seed from the template and make the result's seed-source field a loud, non-silent warning.
Record the prime premise with its failure mode in Auto-mode assumptions.
Keep the template fallback only for `lyx fabric clone`, where no prior vocabulary can exist.

### [LOW] A doc-rewrite detail sits under Auto-mode assumptions

**Location:** `decision-record.md` Auto-mode assumptions, third bullet.
**Issue:** The bullet is plan-level detail about one paragraph of `internal/boardengine/doc.go`, not an assumption about the auto-mode run.
The requirement it carries, that `boardengine/doc.go` stop placing the label list in the per-worktree file, is already covered by the Constraints docs line and the last-but-two Acceptance bullet.
Its "or drops it if the list is already live on the hub" clause is a plan-time check.
**Fix:** delete the bullet from Auto-mode assumptions.
The requirement stays in Constraints; add one clause there, "and the `boardengine/doc.go` label-list paragraph points at the hub file", if the plan writer should not rediscover it.
