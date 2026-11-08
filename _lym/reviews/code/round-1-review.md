---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/unwire_test.go:326
    summary: TestUnwire_LeavesSiblingWorktreeUndirtied no longer asserts which exclude lines the real unseedGitExclude strips, so a wrong anchorRel or names argument at its call site goes unseen
    fix: In TestUnwire_LeavesSiblingWorktreeUndirtied, read the exclude file after the Unwire (excludePath is already resolved in the row) and assert that the unwired pair's anchored line, for example /sub/_lym in row UnwiredPairAloneAtItsAnchor, is gone. Add it as a single wantStripped field or a row-local check, and leave the line-selection logic to TestExcludeLinesToStrip.
---

### [LOW] The real unseedGitExclude wiring lost its exclude-line assertion

**Location:** internal/fabricengine/unwire_test.go:326
**Issue:** The commit moved the kept and stripped exclude-line assertions to the tier 0 `TestExcludeLinesToStrip`, which feeds `excludeLinesToStrip` and `stripExcludeLines` a hand-built `siblingWiring`.
That pins the line choice.
It does not pin the call site in `unseedGitExclude`, which passes `l.AnchorRel` and `names` and builds the census with `namesWiredInSiblingWorktrees`.
The surviving git test compares sibling `git status` before and after, which catches an unwire that strips too much.
It does not catch one that strips too little, such as the unwired pair's own anchored line.
`junction_pattern_integration_test.go:203` asserts only that the bare `_lym` line is gone.
The record allows the move to tier 0 and asks to keep the git test where git is needed, so one stripped-line assertion in the git test restores the lost coverage.

## Rejected

- Lens code-affected-set, MEDIUM scope, `withVacuityCheck` checks once per anchor and not per hub: the record asks for the vacuity check in the fixtures' own apply path, and the commit message and doc comment state the once-per-anchor trade-off, made to stop doubling manifest captures.
  The old `TestStates` also checked each state once per anchor on one resolved target, so the strength per state and anchor is unchanged.
- Lens code-affected-set, open question whether `WarpJunctions` filters `names`: it does not.
  `WarpJunctions` in `junction.go` returns one record per name with `Name: name`, so iterating `names` in `excludeLinesToStrip` selects the same set. The code-correctness lens confirmed it.
- Lens code-tests F1, the removed `TestList` was the only check of the main entry path and a non-empty `Head`: the surviving real-git test at `worktreelist_test.go:190` compares the full entry list, `Path`, `Head` and `Branch`, against a want value from both the repo root and a subdirectory.
- Lens code-tests F3, three removed `TestStates` establishment assertions have no inline replacement: `assertLinkTarget` checks the symlink and its raw target, and `git add -f` followed by `git commit` fails through `gitkit.Git` unless the link is tracked.
  A foreign directory without a `.git`, and a clone with a commit, follow from how the state builds them, with `gitkit.CommitFile` failing on error.
- Lens code-affected-set, LOW consistency, the clean-hub baseline runs at `backend` only: the record asks for the anchor loops cut to the rows a nested anchor needs, so this is no defect.
- Production edits in `dirtiness.go`, `junction.go` and `worktreelist.go`: each only splits a pure step out of the git-running function with the body unchanged, as the record's last constraint allows.
- Tier 0 moves for `TestList`, `TestSourceDirtyPaths` and the exclude strip: no assertion of the removed git-backed rows loses coverage other than the one raised in F1.
- Dropped and edited tests in `reconcile_stale_registration_test.go`: the merged `TestCleanup_VerdictIsIndependentOfDryRunAndForce` keeps every assertion of both originals, including the orphan deletion, the unmanaged branch surviving and the force parity.
  `unwire_test.go` loses only the exclude-line assertions raised in F1.
- `//testtiming:keep` marks: each of the five carries a reason, and the change description names all five as contained tests the module-wide check confirmed.
- Commit messages against the diff: no test is dropped without grounding in the record, and no rarest-block-count-2 test is dropped.
