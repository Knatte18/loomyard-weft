---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/hubforge/template_test.go:167
    summary: no test pins pack-refs --all or the cruft pack, since the packed-template test checks loose objects only on a shape with no unreachable objects
    fix: In TestSharedHub_PackedTemplateHoldsNoLooseObject in internal/hubforge/template_test.go, also collect every regular file under a repository's refs/heads and refs/remotes directories during the same walk, and fail when any exists, since pack-refs --all leaves none. Add that check as an assertion of the existing test, not a new test function.
  - id: F2
    severity: LOW
    class: decision
    origin: handler
    location: internal/hubforge/template.go:269
    summary: packRepositories runs repack -d --cruft where the decision record D3 names git repack -ad, and no recorded parent decision covers the difference
    fix: Leave the code in place and defer this finding to the run's parent. Its only sound change is recording the --cruft choice, or refusing it, with lym loom decision add, which lies outside the fixer's write surface.
---

### [LOW] Nothing pins pack-refs or the cruft pack

**Location:** `internal/hubforge/template_test.go:167`, `internal/hubforge/template.go:261`
**Issue:** `TestSharedHub_PackedTemplateHoldsNoLooseObject` asserts loose objects only.
Deleting the `pack-refs --all` call fails no test, although `doc.go` and the record's D3 promise packed refs.
The test's shape builds no unreachable object, so replacing `--cruft` with `-a` also passes.
The acceptance criteria name loose objects only, so the cruft half is a weaker gap than the refs half.

### [LOW] `--cruft` is an unrecorded departure from the record's `git repack -ad`

**Location:** `internal/hubforge/template.go:269`
**Issue:** D3 in the decision record names `git repack -ad`, and the diff runs `git repack -d --cruft`.
The change description gives the reason: `-ad` leaves unreachable loose objects, so a build that leaves any would still hand copies loose objects.
`--cruft` serves the acceptance criterion "a template's repositories hold no loose objects after the build" better, and `toolfloor.GitFloor` (2.43.0) is above the 2.37 that `--cruft` needs.
No `Pattern-Departure:` trailer is due, because no PATTERN entry forbids either form.
The record carries no `Added after Discussion` entry for the change, so the deviation is unrecorded.

## Rejected

- Lens code-tests: the `requireIntact` branch that detects a change before any cleanup has run is unpinned. The branch is unchanged by the diff and was as unpinned before it, so the diff adds no uncovered behavior.
- Lens code-correctness and lens code-affected-set returned no findings; every case they ran was correct.
- Focus item on `gitDirsUnder`: it finds bares, clones' `.git` directories and, through the main repository, every linked worktree's objects. A `.git` file and a worktree gitdir hold no objects of their own, so no repository is left unpacked. Not a defect.
- Focus item on `commitRelocatedBinding`: `commit --amend -- <file>` stages the file, leaves the board clean, and writes HEAD, branch and tracking-ref reflog entries. `assertBindingFoldedAndPushed` pins the clean board, the SHA equalities and the tracking ref's newest entry. Not a defect.
- Focus item on `requireUnpoisoned` and `requireIntact`: `copyTemplate` checks the poisoned flag only, and `SharedHub` walks the fingerprint before and in its cleanup and poisons on a change. Matches D3.
- Focus item on packing order: packing runs after `requireBoardAtUpstreamTip` and before `fingerprint`. Packs and `packed-refs` hold no template path, and `findRootReference` verifies each copy. Not a defect.
- Focus item on `doc.go`, the file headers and `_lym/landing/summary.md`: every claim checked against the diff holds, and the comments follow the Go comment conventions. Not a defect.
