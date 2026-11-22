---
dispositions:
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed-differently
    reason: "The proposed test, an error raised before the cancel ends anything, cannot be read from the output: `judgeOrigin`'s pre-end errors (`internal/battencli/cancel.go:468`, `:612`, `:620`) carry the same 'fix the cause it names, then run … again' wording as the post-end ones (`:534`, `:550`, `:555`), and `cancel.go:248` is a post-end error the proposed list called an origin-stopped refusal. The two cases are now told apart by the way forward each names; a rerun is safe because every cancel step is a no-op once done (`cancel.go:254-256`)."
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed-differently
    reason: "The proposed 'refused project write leaves the entry a draft with the move note written in' is false for a single upsert that carries both, since a refused upsert writes nothing; the orch now runs the upsert again without the `project` change. The ready record is read on the beam branch, not only in a local worktree, since the new parent's worktree can sit on another machine (`internal/beamcli/cli.go:918-920`)."
---

# Round 2 fixer report

## Changes

All changes are in `_lym/discussion/decision-record.md` and `_lym/discussion/support-log.md`.

- F8, fixed: D2 asks the live writer by `lym creel post --expect-reply` for the first three parts in its reply text, writing no file;
  the orch writes the note in its own `.scratch/`, holds with `lym tender pause <slug>` and adds the fourth part.
  `_lym/move-note.md` and the copy step are gone; D2's departure sentence and Rationale, the Goal, D1's Rationale, D4 step 1, the Auto-mode lines and the D2 acceptance bullet match.
  D3 takes a writer that refuses, or replies without the three parts, as no live writer, and its acceptance bullet says so.
- F9, fixed differently (see frontmatter): D4 step 3 tells a partial finish from a refusal by the way forward each names.
  The refusal list names the hub-pause, owned-elsewhere, reached-its-parent, landed-by-another-machine and nothing-left refusals.
  The orch never runs a slugless `lym tender resume`; on the hub-pause refusal it stops with its slug pause and runs the cancel again once the operator lifts the hub pause.
  D7 and its acceptance bullet carry the hub-pause exception.
- F10, fixed: D4 step 3 confirms both the code tag and no `<task branch>-weft` records branch on origin before step 4, bounded at ten minutes with a manual tag push from the prime.
  When the records branch stays, the orch stops, naming the branch and the note's scratch path, and the move goes on at step 4 once it is gone.
  Step 5 keeps only the pruning fetch.
  D4 step 4's section and D6 name `git fetch origin refs/tags/archive/<slug>/<12 hex>:refs/tags/archive/<slug>/<12 hex>`; the owner Constraint says the fresh run fetches the tag; the D4 and D6 acceptance bullets name both.
- F11, fixed differently (see frontmatter): D4 step 2 reads `_lym/landing/beam-ready.json` on the beam branch, in the local worktree or through `git show origin/<beam branch>:…` after a fetch.
  D4 step 4 runs the upsert again without the `project` change when the board refuses it, keeping the note under the old parent, and tells the operator.
  The D4 acceptance bullet names both.
- Support log: Turn 2's note-place line points to the round 2 change, and a Round 2 section records F8 to F11 and the self-check fixes.

## Self-check

- fix-consistency: no findings.
- fix-proposal MEDIUM, ready record read from a local worktree only (merged with fix-way-forward LOW 3): fixed, read on the beam branch through `git show origin/<beam branch>:…` when the worktree is elsewhere.
- fix-proposal MEDIUM, records branch not named: fixed, `<task branch>-weft` under `refs/heads/` or `refs/lym/heads/`.
- fix-proposal LOW, `lym tender resume <slug>` refused on the owned-elsewhere refusal: fixed, D7 has the orch tell the operator, who chooses between the adopt and the cancel on the owner.
- fix-proposal out-of-lens note, the records-branch stop leaves the note only in `.scratch/` (merged with fix-way-forward MEDIUM 2): see below.
- fix-way-forward MEDIUM 1, partial finish and refusal not told apart by output (merged with fix-claims MEDIUM 1): fixed differently from the review's F9 proposal, by the way forward each error names.
- fix-way-forward MEDIUM 2, records-branch stop: fixed; the stop names the branch and the note's scratch path, and the move goes on at step 4 once the branch is gone.
  Its problem 4, a body upsert never probes the records branch, is fixed differently: the check stays for every entry, and D4 step 3 now says why, since a fresh run would adopt a live records branch (`internal/boardengine/recordupsert.go:44`).
  Its problem 3, a tag-covered leftover reads not live, is fixed differently: the stricter check stays, since such a leftover arises when the origin delete failed, which mails a notice whose way forward the orch takes before checking again.
- fix-way-forward LOW 4, no step after the operator lifts the hub pause: fixed, the orch runs the cancel again.
- fix-claims LOW 2, no "origin-stopped refusal" exists: fixed, dropped from the list; `cancel.go:248` is a partial finish.
- fix-claims LOW 3, the plan writer has no stated write limit: fixed, the D2 Rationale names the limits of the Discussion writer and the Webster master alone, and rests the reply channel on needing no file write.
- fix-claims LOW 4, the notice names `git -C <worktree>`, not the prime: fixed, the manual push runs from the prime because the worktree is removed.

## Disputed

None.

## Deferred

None.

## Rulings

None.
