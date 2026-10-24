# Round 2 fixer-report — discussion

Every finding's premise held against the code it cites: `internal/battencli/startguard.go:78-85`, `internal/fabricengine/add.go:188-193`, `internal/fabricengine/origin.go:139-157`, `internal/boardengine/owners.go:118-125`, `internal/hubgeom/parent.go:83`, `internal/boardengine/layer.go`, `internal/landingshed/doc.go` and `internal/agentname/agentname.go:102-115`.

## Changes

All in `_lyx/discussion/decision-record.md` unless noted.

- F1 (BLOCKING), fixed as proposed, then widened at self-check: D11 lets a beam's landing past the start guard's left-pair refusal, and its D11 criterion covers a beam from `create` and from `promote`.
- F2 (MEDIUM), fixed as proposed: D5 says that with `Parent` named, Add refuses no tracked dirt in either the parent or the acting worktree, and with it empty keeps today's refusal; D7's criterion adds a member started over a dirty prime.
- F3 (MEDIUM), fixed as proposed, then refined at self-check: D4 says a found record's parent slug is its `ParentWorktree`, never matched by `ParentBranch` against the pair worktrees present here; D4's criterion adds a member whose beam pair is absent here.
- F4 (MEDIUM), fixed as proposed, plus a way forward added at self-check: D8's bound refuses the guard in a beam its landing holds, naming the landing; D8's criterion adds it.
- F5 (MEDIUM), fixed as proposed, then refined at self-check: D1 gives an open project the layer `Project`, never Ready; D1's criterion adds it.
- F6 (LOW), fixed differently: D11 gains a Bound naming the conflict session's resolution as what lands unreviewed.
  The proposed wording, "bounded by the `finalize` gate ... and by the fixer's size and kind caps", was carried out first, and then replaced at self-check.
  The caps judge `LineChange(Baseline, HEAD)` from the HEAD before the fixer's first turn (`internal/landingshed/doc.go:43`), which already holds the resolution, so only the gate bounds it.
- F7 (LOW), fixed as proposed: D11's first refusal reads `ParentOf`, not `ParentBeam`.
- F8 (LOW), fixed as proposed: D10's criterion says steps 2, 3 or 4 and a re-run after any step.
- F9 (LOW), fixed as proposed, then reworded at self-check: D3's Bound admits the create only on the effective owner's machine.
  The D3 criterion twin gets the same change.
- F10 (LOW), fixed differently: the proposal sent the ready notice to the prime's orch.
  The self-check showed that orch's `parent_beam` filter (D8) keeps it from starting the task.
  Batten would also refuse the start under the beam's owner.
  So D13 now names such a task in no notice, with a Bound that no landing hands work between machines.
  D13's criterion adds it.
- F11 (NIT), fixed as proposed: every `ly:<slug>:orch` and `ly:<beam>:orch` in Scope, D8 and the D8 and D13 criteria is now `<shortname>:...:orch`.
  The twins in `_lyx/discussion/support-log.md` (setup notes, Turn 4 and Q7) get the same change.
- `_lyx/discussion/support-log.md` gains a `### Round 2` entry summarising the picks.

## Self-check

- fix-consistency, fix-way-forward and fix-proposal, merged; MEDIUM; fixed: the start guard's tracked-dirt refusal still blocked a landing over a dirty prime.
  D11 now exempts the landing from it too, and Finalize's dirty-parent Stuck guards the worktree it merges into.
  D7 names the landing exemption, and D11's criterion adds a dirty prime.
- fix-claims; MEDIUM; fixed: the F6 Bound claimed that the fixer caps bound the resolution.
  It now names the gate alone.
- fix-way-forward, MEDIUM, and fix-proposal F4, LOW, merged; fixed: the D8 landing refusal named nowhere to run the verb.
  It now names the beam's parent worktree, and the D8 criterion follows.
- fix-way-forward, MEDIUM, and fix-proposal F10, LOW, merged; fixed: the D13 notice went to an orch barred from starting the task.
  Such a task is now named in no notice, the leave-it-out option both forks offered.
  The other option, a report-only notice, would add a stencil rule no decision covers.
- fix-proposal F3; MEDIUM; fixed: `ParentWorktree` is a machine-local directory name (`internal/fabricengine/add.go:441`).
  D4 now reads the prime when `ParentBranch` is the prime's branch or `ParentWorktree` names the prime's worktree.
  D4's criterion adds a prime with a different directory name.
- fix-consistency #2; LOW; fixed: D5's "checks no tracked dirt" conflicted with the Info log of dirty paths.
  It now says "refuses no tracked dirt".
- fix-consistency #3 and fix-proposal F5, merged; LOW; fixed: a project held by its landing lists as `Running`, in D1 and its criterion.
- fix-consistency #4 and fix-proposal F9, merged; LOW; fixed: D3's Bound and criterion now defer to D9's refusal of a beam another machine owns, which removes the mixed `<slug>` placeholder.
  D9's admission now reads by effective owner (D4).
- fix-way-forward; LOW; fixed: a beam pair with a missing records worktree has no readable origin record, so it fell back to `cancel --remove`.
  For a `beam: true` project, D11's left-pair refusal now names `lyx fabric reconcile`, which recreates a missing weft worktree (`internal/fabricengine/doc.go:625`).
  D11's criterion adds it.

Every fork returned a report.

## Disputed

None.

## Deferred

None.
