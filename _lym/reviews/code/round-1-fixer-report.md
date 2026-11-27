---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: "Rendering \"\" for scratch files alone drops a spawn killed before its first commit, with a half-finished measurement in .scratch. The decision record's Goal asks for the prior work to come from the commits and the surviving scratch files. Scratch entries with no commit and no outcome are now listed with a note that claims no interruption and names another agent as a possible author."
  - id: F2
    disposition: fixed-differently
    reason: "The proposed first sentence says any non-(none) prior work is what an earlier spawn left, which is false for the scratch-only note. The stencil says it names what is already on the branch or in the scratch directory. It also drops \"which you continue from\" from the lead-in."
  - id: F3
    disposition: fixed-differently
    reason: "The proposed fixture, hubLocation, is no git repository (internal/loomcli/hubfixture_helper_test.go:76), so no commit can be made on it. The test is TestDarnDeps_BranchWorkSeams over a real pair in subjects_integration_test.go (tier 1)."
  - id: F4
    disposition: fixed-differently
    reason: "A list of modules breaks the format: contracts/specs/loom-final-summary-spec.md:42 requires one word. The blast radius is now darn, which covers the row, its wiring and its registry entry."
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed-differently
    reason: "F7's change stops the listing from descending, so there is no per-entry walk error left to skip. A failed read of the scratch directory itself still halts, now with a test row, beside the dangling-link and unreadable-subdirectory rows."
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed-differently
    reason: "The proposed wording says PriorWork names work an earlier spawn left, which the scratch-only note does not claim."
  - id: F9
    disposition: fixed
---

# Fixer report, code round 1

## Changes

- F1, F5, F6, F7, F9 (`dbd885176`):
  - `renderPriorWork` claims an interrupted spawn only when commits beyond the base exist. Scratch entries alone get a note that claims none.
  - After a recorded outcome with anything to list, the note says a later spawn may have been interrupted.
  - `listScratchFiles` became `listScratchEntries`, which lists the direct children only, directories with a trailing slash.
  - The doc comments, `doc.go` and the tests follow, and the recorded-outcome row now carries a scratch entry.
- F2 (`477bba4bd`): rewrote Step 2 of the darn writer's stencil. Fixed differently: the proposed first sentence was not used.
- F3 (`05dafc7bb`): added `Darn.BranchCommits` to `darnSeams` and added `TestDarnDeps_BranchWorkSeams`. Fixed differently: the test is not on the hermetic fixture.
- F4 (not committed, as a run record): the blast radius is `darn`. Fixed differently: the modules are not listed. The change description's body also follows the F1, F5 and F7 behaviour and the new tests.
- F6: fixed differently: no per-entry skip was added to a walk, since the walk is gone.
- F8 (`28b6d6d52`): rewrote the `DarnInputs.PriorWork` doc. Fixed differently: the proposed wording was not used.
- Self-check fixes: `7e5dd6aba` and `e96571fca`.
- Comment wrapping: a check after the round found comment lines broken mid-sentence in `internal/darnengine/spec.go`, `internal/loomcli/subjects_integration_test.go`, `internal/loomshed/darnwrite.go` and `internal/loomshed/doc.go`. Each is rewritten as whole sentences, one per line, in a commit of its own. No code changed.

`lym gate test --wait --tier 0 --tier 1` over `internal/loomshed`, `internal/loomcli` and `internal/darnengine` passed with no failure after the last change. The same run with `contracts/stencils` added passed before the comment wrapping, which touched nothing there. `lym loom validate-description` is clean.

## Self-check

- fix-code-correctness, LOW, the wiring test cannot tell the task's merge base from the pair's: fixed. The test now merges an unreached parent tip and names it in the merge record.
- fix-code-correctness, LOW, directories listed under text calling them files: fixed, merged with the next two wording findings. Both strings now say entries.
- fix-consistency, LOW, the same "files" wording over entries: fixed.
- fix-consistency, LOW, the change description's stencil bullet reads as against its design decision on commit authorship: fixed, the bullet is reworded.
- fix-proposal, LOW, an empty subdirectory counts as surviving work: fixed differently. The note now says entries, so it holds for an empty directory. The directory stays listed, because dropping it needs a read of each subdirectory.
- fix-proposal, LOW, the later-spawn line also appears on ordinary resumes and the stencil sentence states more than the note: fixed differently. The stencil sentence is reworded. The line stays hedged, because an ordinary resume and one after an interrupted spawn leave the same data.
- fix-affected-set: no findings.

## Disputed

## Deferred

## Rulings
