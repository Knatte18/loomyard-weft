```yaml
lens: code-correctness
outcome: reported
```

List: code-correctness lens, one case run per function or branch.

- `packRepositories`:
  - **Empty bare:** correct. `git repack -d --cruft --quiet` and `git pack-refs --all` both exit 0 on an empty bare repo (checked with git 2.53.0).
  - **Repo with an amended-away commit:** correct. After the repack, no loose objects were left.
  - **Git floor:** `--cruft` needs git 2.37, and `toolfloor.GitFloor` is 2.43.0, so every supported git has it.
- `gitDirsUnder`:
  - **Bares and clones:** correct. A bare repo and a clone's `.git` directory both hold `HEAD` and `objects`. Each is returned once, and `SkipDir` stops the walk inside it.
  - **Linked-worktree gitdirs:** correct. `.git/worktrees/x` is never reached because its parent was skipped, and it has no `objects` of its own anyway.
  - **`.git` file in a linked worktree:** correct. A file is not a directory, so it is ignored. Its objects live in the main repo, which is packed.
  - **Walk error:** correct. It is returned and `packRepositories` calls `tb.Fatalf`.
  - **Junctions on Windows:** not verified. `WalkDir` may report a junction as a directory there. This is an observation only, not a finding.
- `requireUnpoisoned`: a poisoned template fails with the fixture name. Correct.
- `requireIntact`: calls `requireUnpoisoned`, takes the fingerprint, and on a mismatch sets the poisoned flag and fails. Correct. `SharedHub` still checks before the test and again in its cleanup.
- `copyTemplate`: checks only the poisoned flag, as the record asks. Correct. A stray write by a `SharedHub` test before its cleanup is the bound the record accepts.
- `commitRelocatedBinding`:
  - **Binding file changed by the relocation:** correct. `commit --amend --no-edit -- <file>` staged and committed the file, and `git status --porcelain` was empty afterwards.
  - **Reflog entries:** HEAD, branch and tracking ref get reflog entries as for any commit and push.
  - **Binding unchanged:** an amend with a pathspec still succeeds. Correct.
- `buildTemplateIn`: packing runs after `requireBoardAtUpstreamTip` and before `fingerprint`. Correct. Any `git` that touches the template afterwards would be a `SharedHub` write.
- `assertBindingFoldedAndPushed`: every call is valid on a packed template. `reflog show` on a packed tracking ref still works, because `pack-refs` leaves `logs/` alone. Correct.
- `TestSharedHub_PackedTemplateHoldsNoLooseObject`: the loose-object detector matches `objects/XX/<file>` with `XX` two hex digits.
  - **Shape clash:** no other test in the `hubforge` test binary takes the backend-with-pair shape through `SharedHub`.
  - **Gap:** the test asserts loose objects only, not loose refs. This falls under the tests lens and is not reported here.

No correctness findings.

Tool calls:
1. Ran a first scratch-dir check and the hook refused it, since its `rm` was on a computed path.
2. Ran the same check in a scratch directory outside the repo: `repack -d --cruft` and `pack-refs` on an empty bare, and `commit --amend -- file` followed by a cruft repack. My context could not show how git behaves.
3. Ran `git --version` earlier in the parent turn (2.53.0), and read `toolfloor.GitFloor` (2.43.0) for the `--cruft` version check.
