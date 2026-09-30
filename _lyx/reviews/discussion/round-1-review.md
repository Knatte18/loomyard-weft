---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md (Notes for the plan writer; Open risks; D6 rationale)
    summary: The note confines the change to remove.go, doc.go and remove_*_integration_test.go, but two tests outside those files assert the old order outright and will fail under D1, while the battencli open risk does not materialise
  - id: F2
    severity: LOW
    location: _lyx/discussion/decision-record.md (Goal, third sentence)
    summary: The Goal says any refused Remove leaves hub and weft origin untouched, but gate refusals surfaced from the portal, launcher and junction teardown and a failed warp worktree removal still follow the archive under D1
  - id: F3
    severity: LOW
    location: _lyx/discussion/decision-record.md (Constraints, byte-identical bullet; D3; Acceptance criteria 1-2)
    summary: The byte-identical constraint misdescribes the livestate check and reads as contradicting D3, which drops the reconcile suffix; the no-reconcile acceptance check covers only the warp refusal
  - id: F4
    severity: NIT
    location: _lyx/discussion/decision-record.md (D2 rationale)
    summary: D2 carries commit archaeology (9de0c46d6, the header's history) the plan writer does not need; the support log's Interview turn 3 already holds it
  - id: F5
    severity: NIT
    location: _lyx/discussion/decision-record.md (Constraints, last bullet)
    summary: The semantic-line-break bullet restates a repo-wide CLAUDE.md rule and is not a constraint of this task
---

### [MEDIUM] Confinement note is false and the open risks are stated in the wrong direction

**Location:** `decision-record.md`, `## Notes for the plan writer` first bullet, `## Open risks`, and D6's closing rationale ("the remaining tests pinned the old order").

**Issue:** The note tells Plan-Write the change "is confined to `internal/fabricengine/remove.go`, its doc passage in `doc.go`, and the `remove_*_integration_test.go` files named in D6", and D6's rationale reads as if D6 lists every test pinning the old order.
Plan-Write reads only this file, so the note is the scope picture it plans from.
Two tests outside that set assert the old order directly, not by accident:

- `internal/fabricengine/livestate_verbs_test.go`'s `removeCase` expects every dirty-refusal cell (`dirtyWarpTracked`, `dirtyWarpUntracked`, `dirtyWeftTracked`, `dirtyWeftUntracked`, `bothDirty`) to permit the portal and launcher roots and asserts `assertGone` on the portal link and launcher dir after the refusal.
- `internal/fabricengine/mutation_record_integration_test.go`'s `TestMutationRecord_RemoveDirtyWarpRefusalCarriesThePortalAndLauncherDeletions` asserts the refusal's record carries `KindLinkRemoved` and `KindPathRemoved`; its whole premise is the ordering defect this task removes.

Both fail under D1, and each needs an expected-outcome decision (invert or retarget), not just a green run.
`## Open risks` frames this as "may encode", while the `battencli` risk it lists beside it does not hold: no `internal/battencli` integration test drives a dirty refusal, and `remoteArchiveTagExists` is asserted only after teardowns that archived and removed the pair.

**Fix:** Drop the confinement bullet.
Add a D6 bullet deciding the two tests' new expectations: the livestate `Remove` dirty-refusal cells assert the portal, launchers and warp worktree survive, with no portal or launcher permitted roots;
the mutation-record test's `Remove` case asserts the refused call's record is empty and `RefusalOf` still reports false.
Replace the two open risks with the remaining real uncertainty (other integration tests driving a dirty `Remove` for another purpose surface only in the full run).

### [LOW] The Goal overstates what a refusal leaves untouched

**Location:** `decision-record.md`, `## Goal`, "After this task a refused `Remove` leaves the hub and the weft origin exactly as it found them".

**Issue:** D1 keeps the portal and launcher teardown, the junction sweep and the warp worktree removal after `archiveWeftTip`.
Each can still return a refusal after the tag is pushed: `surfaceRefusal(removePortal(...))`, `surfaceRefusal(removeLaunchers(...))`, `surfaceRefusal(removeWarpJunction(...))`, and `removeWarpWorktreeDir`'s gate refusal.
Those are exactly the "later teardown step failed" states D6's reuse test models, so the Goal's unqualified "a refused `Remove`" contradicts D6 and could send Plan-Write hunting for a reorder the design does not want.

**Fix:** Scope the sentence to refusals from the checks that precede the archive (the dirtiness checks and their status probes included).

### [LOW] The byte-identical constraint is misdescribed and reads against D3

**Location:** `decision-record.md`, `## Constraints` fourth bullet; D3; `## Acceptance criteria` first two bullets.

**Issue:** The livestate refusal matrix does not compare `Remove`'s message byte-for-byte against `destroy.go`'s.
`RefusedBefore` (`livestate_refusal_test.go`) matches a substring and excludes any message containing "check failed", precisely because `destroy.go`'s gate dirtiness reason and `Remove`'s pre-flight text are the same string.
Today a first refused attempt appends the reconcile remedy, so after D3 the full message is not byte-identical to today's; only the base texts are.
The bullet as written can be read as forbidding the change D3 makes.
Separately, the acceptance criteria require "does not mention `lyx fabric reconcile`" only for the warp refusal, while D3 drops the suffix from both.

**Fix:** Restate the constraint as: the base refusal texts (`worktree has uncommitted changes; use --force`, the weft sibling text, the `check warp/weft worktree status` wrappers) stay unchanged, because the livestate `RefusedBefore` check relies on the pre-flight text matching the gate's dirtiness reason; only the reconcile suffix goes.
Make the no-reconcile check apply to both dirty refusals.

### [NIT] D2 carries commit archaeology

**Location:** `decision-record.md`, D2 rationale, "That rationale dates from `9de0c46d6` …".

**Issue:** The plan writer needs to know the header's justification is stale and why (a missing worktree dir now returns "worktree not found" before any teardown), so it can rewrite the header and doc comment.
The commit hash and the history of where the teardown used to sit are deliberation narrative; the support log's Interview turn 3 already records them.

**Fix:** Keep the stale-rationale fact and the reason it is stale; drop the commit reference and the history sentence.

### [NIT] Generic semantic-line-break constraint

**Location:** `decision-record.md`, `## Constraints`, last bullet.

**Issue:** "Comments and markdown follow the repo's semantic-line-break rule" is CLAUDE.md's repo-wide rule, binding on every task; it carries no requirement specific to this one.

**Fix:** Remove the bullet.

## Focus verdicts

- **D1/D2 reorder soundness:** sound.
  The portal link lives under `_portals` and the launcher dir under `_launchers`, both hub containers outside the warp and weft worktrees, so removing them cannot change either `git status --porcelain` probe.
  Under `force` the dirty checks are skipped and the portal and launcher teardown still runs after the archive, as today.
  A `surfaceRefusal` from `removePortal`/`removeLaunchers` still follows the archive and leaves a tag; that is by design under D1 but contradicts the Goal's wording (F2).
- **D3 vs byte-identical:** the base texts stay identical and `RefusedBefore` still matches; the constraint's wording is the defect (F3).
- **Notes vs Open risks:** the note overstates confinement (F1).
- **D6 reuse test:** not a defect.
  After the reorder, a same-tip tag is left on origin and locally by any failure after `archiveWeftTip` (a portal, launcher or junction gate refusal, a failed warp worktree removal, a weft teardown failing with the weft worktree still present), and by refusals under the old order.
  `archiveWeftTip` reuses a tag only when it exists locally at the tip, so D6's "on origin and in the weft repo" seed is the right state.
- **Leanness:** F4 and F5.
