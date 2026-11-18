---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed-differently
    reason: "Lacking is read from origin's ref of the branch being pushed, not from any origin ref, since any-ref reachability lets the guard push operator commits fast-forwarded from a pushed feature branch; a merge fabric wrote is identified by its SHA recorded in .lym, not by message."
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
  - id: F14
    disposition: fixed
  - id: F15
    disposition: fixed
  - id: F17
    disposition: fixed
  - id: F18
    disposition: fixed-differently
    reason: "Stopping as owed whenever origin added code would halt every pull-request landing with unpushed parent records, undoing F2. Instead a replayed Warp-SHA that names a replayed commit is rewritten to the fork point, whose code the commit was made over, so it never names code it never judged."
  - id: F19
    disposition: fixed
  - id: F20
    disposition: fixed-differently
    reason: "The class deferred sends no creel notice (internal/fabricengine/failedpush.go:170) and means held back, not refused; the guard's refusal gets a new class operator_unpushed that FailedPushesOwedBy reads as owed and whose notice is sent."
  - id: F21
    disposition: fixed
  - id: F22
    disposition: fixed
  - id: F23
    disposition: fixed
---

# Fixer report — discussion round 1

Changes are all in `_lym/discussion/decision-record.md`, plus a round-1 entry in `_lym/discussion/support-log.md`.

## Changes

- F1: D8 makes the publish idempotent. It reuses origin's view tip when the tree and parent match, writes a commit with fixed identity and dates otherwise, and pushes again when origin lost the ref. Added a D8 criterion row.
- F2: D5's fast-forward sentence became the level-with-origin rule: commit records, replay records-only commits, and refuse only on an unpushed non-records commit. D8's post-merge step uses that rule. D9 adds the parent-tips relation as a code-only reader. Added a D8 criterion row for landings with unpushed parent records.
- F3: D2 refuses a protected marker push and names the protection. The protected-branch match moves into fabric's push classification. D7 adds a `protected` failed-push class that holds no run, plus a bound. Added D2 and D7 criterion rows.
- F4 (fixed differently, see frontmatter): D7 defines the records-only, fabric-merge and lacking predicates, and states that a message alone never admits code. D5 step 4 records the merge SHA in `.lym`. The guard admits commits a fabric merge brings in through its second parent.
- F5: D6 says the owed-code-tip push, the given-up branch delete and `ReturnToPreviousTransition` do not run in a one-repository hub. Added a takeover criterion row.
- F6: D8 has a rule that every pull request head site uses the published view tip, and that a merged pull request holds the work by view identity or D9's check, never by ancestry. Added a criterion row.
- F7: D4 adds a bullet for empty records commits with code staged. Added a criterion row.
- F8: D9 adds the allow-list walk kind (`reconcileHead`). Added a criterion row.
- F9: D5 defines records paths as the union of a commit's own marker and its parent's, and D7 and D9 read them that way. A merge whose source edited the marker refuses, with a way forward. Added criterion rows.
- F10: D5 step 3 compares the restored paths against what step 2 wrote and refuses on a change. The Bound now covers steps 2 to 3. Added a criterion row.
- F11: D3's Add drops the parent's run-records root. Added a criterion row.
- F12: D7 records the prime's branch in `.lym` and refuses records commits off it, leaving the records on disk. The way forward names `git switch --merge`. The bounds cover a stall while the prime is off its branch, and reset and stash. D2's docs sentence carries the warning. Added a criterion row.
- F13: rewrote the plan-writer note to strip the base and both sides before the in-memory merge. Added a D9 criterion row for a squash landing.
- F14: D2 orders the marker push before `lym-bolt` and pushes nothing after a refusal. A refused clone removes its hub directory. A one-URL clone with no marker refuses and names a way forward. Added criterion rows.
- F15: added a Constraint that pins each tool fact. D4 is pinned by `git-commit --only` and `TestCommitPathAllowEmpty`'s row `UnchangedPathWithAnotherPathStaged`. D5 is pinned by the `git-merge` and `git-restore` docs plus its acceptance scenario, D8 by GitHub's three-dot diff docs plus its scenario, and D6 by GitHub's automatic branch-deletion docs.
- F17: added a `PATTERN-cwd-resolution` changes bullet and dropped that pattern from the Constraint's list of overview lines that stay.
  A gate run then found the bullet malformed, because it wrapped onto a second line; it is now a single line.
- F18 (fixed differently, see frontmatter): D7's trailer rule is rewritten. The replay never stops just because origin added code. Updated the D7 criterion.
- F19: D7 now says "on any code branch", and `lym-bolt` keeps the by-entry Bolt replay.
- F20 (fixed differently, see frontmatter): D7 names the `operator_unpushed` class and the prime runs it lists. Added it to the named-constants Constraint.
- F21: D8's verb takes the pull request's target as the base and refuses a mismatch with `parent_branch`, with a way forward. A push-time refresh uses `parent_branch`. Added a criterion row.
- F22: the Auto-mode assumptions drop "the mode built now" and add D5, D7, D9 and D11.
- F23: extended the D4 criterion (one write lock) and the D7 criterion (hold until the push lands).

## Self-check

- fix-consistency, BLOCKING: D5's leveling conflicted with F18's stop and re-broke F2. Fixed differently (F18 entry above): rewrite the trailer to the fork point, with no stop.
- fix-consistency, MEDIUM: the view refresh at push time had no base. Fixed as proposed: it uses `parent_branch`.
- fix-consistency, LOW: D2's docs sentence lacked the warning D7 cites. Fixed as proposed.
- fix-consistency, LOW: D5 step 4 did not record the merge SHA. Fixed as proposed.
- fix-consistency, LOW: "both its parent and itself" was ambiguous next to the union. Fixed as proposed: D7 now cites D5's records paths alone.
- fix-claims, MEDIUM: `protectedBranchRefusal` is unexported in `landingshed`, which fabric cannot import. Fixed as proposed: the match moves into fabric's push classification, and `landingshed` reads it.
- fix-claims, LOW: `deferred` means held back, not refused. Merged with fix-proposal's `deferred` finding below.
- fix-proposal, BLOCKING: F18 undoes F2. Merged with the fix-consistency BLOCKING finding above.
- fix-proposal, MEDIUM: any-ref reachability lets the guard push operator commits. Fixed as proposed: reachability is read from origin's ref of the branch, and a fabric merge's second-parent commits are admitted.
- fix-proposal, MEDIUM: `deferred` sends no notice. Fixed differently: new class `operator_unpushed`, read as owed, notice sent.
- fix-proposal, MEDIUM: an idempotent publish did not recreate a view ref that origin lost. Fixed as proposed: the comparison is against origin's fetched view ref, and the publish pushes when origin lacks it. Added a criterion row.
- fix-proposal, LOW: the off-branch refusal stalls prime runs with no stated bound. Fixed as proposed: a bound plus the docs warning.
- fix-way-forward, MEDIUM: a protected prime branch with operator commits unpushed still holds prime runs for good. Deferred, see below.
- fix-way-forward, MEDIUM: the missing-marker clone refusal had no way forward. Fixed: the way forward commits the marker that `lym-bolt` names, then clones again.
- fix-way-forward, LOW: a rerun after a refused clone would meet "hub already exists". Fixed: a refused clone removes its hub directory, and the non-protected refusal names a way forward.
- fix-way-forward, LOW: the D5 marker refusal and the D8 base refusal had no way forward. Fixed as proposed for both.
- fix-way-forward, LOW: `git switch` back can refuse the uncommitted records. Fixed: the records stay on disk, and the way forward names `git switch --merge`.

Every fork returned a report.

## Disputed

None.

## Deferred

- Self-check, fix-way-forward MEDIUM: a prime branch that refuses direct pushes, with operator commits unpushed on it.
  The guard refuses before any push, so it never learns of the protection. The `operator_unpushed` hold then waits on a `git push` that origin refuses.
  Every fix needs a design call this round cannot make alone. One option releases the hold on such a branch, which gives up cross-machine safety of prime transitions. Another has a lym verb move the operator's commits off the prime's branch while keeping fabric's records commits. A third pushes the prime's records to a side ref.
  This goes to the parent with the next round.

## Rulings

None reached this round.
