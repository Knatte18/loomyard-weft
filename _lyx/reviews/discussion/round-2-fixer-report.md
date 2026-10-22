# Round 2 fixer report — discussion

## Changes

- F1 (BLOCKING), fixed as proposed.
  D2 decides to widen `contracts/specs/refusal-spec.md`'s way-forward item 3 in the same commit, to plain `git` in a checkout the message names when the work there is the operator's and no `lyx` verb covers it, and states that D2, D4 and D5 rely on it.
  The Constraints bullet on `PATTERN-refusal-way-forward` names the amendment; a Scope bullet and an acceptance criterion were added.
- F2 (MEDIUM), fixed differently.
  D2's "reaches origin with Finalize's parent push, with no gate run" sentence is replaced with the route the code takes: step 1b2's `parentGuard.moved` reads the commit as moved and Finalize reroutes to Merge-In.
  The proposed route, in which the commit is merged into the task branch, was not adopted as written.
  Merge-In's `syncParent` (`internal/landingshed/mergein.go`) stops first on a local parent ahead of its upstream with "push or reconcile … in its own pair", unless the push is skipped.
  So D2 states that stop, then the merge, Merge-Bouncer's review perch, Publish and PR-Gate, and that a discard lands directly.
  The proposed acceptance criterion was added: a commit reroutes to Merge-In with no dirty halt, and a discard lands.
- F3 (MEDIUM), fixed as proposed, extended by the self-check.
  D2's merge-in-progress arm names the parked merge's verb and source, read with `fabricengine.MidMerge` on the parent pair.
  Its way forward names `lyx fabric merge --continue` / `--abort` in the parent pair, run by the party that started the merge, or by the orch or the operator when that party is gone, then the goto and the resume, ending with the role-guard clause.
  It also warns that `--abort` hard-resets the parent's code checkout, so the operator first sets aside their own changes there.
  The acceptance criterion was updated, and a plan-writer note was added.
- F4 (LOW), fixed as proposed, extended by the self-check.
  When the guard error carries dirt plus other reasons, D2 keeps the full text and the path list.
  Its way forward names the dirt step first, then Merge-In's step for each parent-side reason (not synced, detached head), then the goto and the resume.
  A source reason gets no step of its own, since it concerns the task branch.
  An acceptance criterion was added.
- The D4 merge-record sentence now says no one is told to commit or discard another merge's dirt with plain git.
- The support log gained a Round 2 section.

## Self-check

- fix-consistency, LOW: "One function renders" and D4's refusal implied that D4 and D5 render every variant of D2's way forward. Fixed as proposed: both now name the dirt-only rendering.
- fix-claims, MEDIUM, merged with fix-proposal BLOCKING: `--abort` hard-resets the code checkout and discards the operator's dirt, and it refuses on `merge conclude already landed`. Fixed as proposed: the "stale record is cleared by the same `--abort`" claim was replaced with the reset warning and the conclude-landed refusal.
- fix-claims, LOW: no row is named Merge-Review. Fixed as proposed: it is now Merge-Bouncer's review perch.
- fix-way-forward, MEDIUM, merged with fix-proposal MEDIUM: "clears the other named reasons" named no command, and a source reason is not parent-side. Fixed as proposed: D2 names a concrete step per parent-side reason and bounds the source reasons.
- fix-way-forward, LOW: for a stale record, nobody was named to run the verb. Fixed as proposed: the orch or the operator runs it when the starting party is gone.

## Disputed

None.

## Deferred

None.
