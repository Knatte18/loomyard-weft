# Cut fabricengine tier 1 tests from the coverage study's removable set

Drops the `internal/fabricengine` tests in the removable set of the coverage study,
`greedy.tsv` in `docs/research/speed/fabric-test-redundancy-2026-11-08/`.
The set is the tests that can go together without leaving a package-scoped block uncovered, slowest first.
The earlier cut (`f2ac2bf7`) was vetoed almost entirely by the module-wide `go run ./cmd/testtiming -redundancy` check,
which counts the fixture packages' blocks too.
This cut treats the study's list as the only authority and does not consult that check.
Production code does not change.

## What changes

- 188 `Test` functions are removed across the `internal/fabricengine` test files, with the helpers and `export_test.go` shims
  (`WorktreeDirtyTrackedForTest`, `WorktreeDirtyAllForTest`) that no remaining test calls.
- 64 tests in the set are kept instead, each marked `//testtiming:keep <reason>` naming what it pins.
  The reasons fall into these groups:
  - Archive tags: their names, targets, content and order, including the cases where no tag may exist
    (rolled-back Add, no origin, refused Remove, dirty worktree).
  - Deletes on origin: a branch removed on origin under `remote=true`, and left intact under `remote=false`, a dry run or a held lock.
  - User content never claimed or removed: stray or look-alike directories, uncommitted edits, hand-authored symlinks,
    the operator's exclude pattern and the primary weft branch.
  - Mutation records and their order, including the cases where an entry must be absent
    (`repo_advanced`, `remote_branch_deleted`, `remote_branch_updated`, `worktree_reset`, `file_written`, `commit_created`, `link_removed`).
  - Machine-local files kept out of the weft tree or out of git status (scratch, merge-state record, lock directories).
  - Merge and abort safety: commits landed while MergeIn or MergeAbort waited surviving, recorded start SHAs, a conflicted weft left untouched.
- Several tag-asserting tests the first pass dropped are restored and marked as keeps,
  and the `remote=false` archive case is folded into the happy-path archive test.
- `cmd/lym/cwdmutation_test.go`: the coalesce test no longer changes the working directory,
  so its `cwdMutationAllowlist` entry and the allowlist half of `TestCwdMutationGuard_NotVacuous` go.
  The coalesce file stays in the guarded subject set with an updated reason.

## Design decisions

- The default was to drop.
  A test is kept only when it asserts something concrete that block coverage cannot see.
- Only tests in the study's set were touched, so every rarest-block partner stays.
- Fabricengine test file headers were rewrapped to one sentence per line.

## To check by hand

- The tier 1 time of `internal/fabricengine` before and after this change was not captured here.
  Run `lym gate test --tier 1 ./internal/fabricengine` on this branch and on `main` on the same machine to record both times.
- Review the kept tests' `//testtiming:keep` reasons for any that block coverage already shows and could go.
