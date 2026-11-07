```yaml
lens: fix-way-forward
outcome: reported
```

Task: check, through the fix-way-forward lens, that every refusal, step, row and test this fix introduced can be carried out and leads somewhere.

Ways forward the fix introduced:
1. D1 Bound: on the reader's named reftable error, both callers fall back to `git worktree list --porcelain`. `List` falls back to its present parse, and `internal/websterengine/gitwrap.go` to its present scan of the `worktree` lines.
2. D1 floor paragraph: the relative-`gitdir` parity test skips below git 2.48 and names the version.
3. D2 TB bullet and criterion: `Chdir` and `ArtifactDir` fail the build, naming the key.
4. D2 Bound and the new D7 Review criterion: review refuses a build that calls `os.Setenv`, `os.MkdirTemp` or `git config --global`, or keeps the TB's `Context()`.
5. D7 stay-in-test rule and the D2 criterion: a step that leaves the board's HEAD unequal to its upstream tip, ahead or behind, stays in the test after the copy.
6. D8: `TestNoticeSite` stays at tier 1, and D9 task 11 drops the move.

Items 1, 2, 3, 5 and 6 can each be carried out from the state they name:
- Item 1: `otherWorktrees` (`gitwrap.go:323-347`) already holds the porcelain path the fallback keeps.
- Item 2: `internal/toolfloor` already parses `git version`.
- Item 3: `Chdir` and `ArtifactDir` are methods of `testing.TB` under go 1.26 (`go doc testing.TB`).
- Item 5: the ahead case, `SeedFabricConfig`, can run on the copy.
- Item 6: needs no further step.

Findings:

- LOW. Concerns the D2 Bound sentence "Review bounds them: the review of each registration in its D7 task checks that its build makes none of those calls", with the new D7 Review criterion. Summary: the review refuses a build step that calls `os.Setenv` or `git config --global`, or keeps `Context()`, but D7 gives that step nowhere to go. Evidence: D7 says builders are "the fixture's present build steps, moved as they are". Its stay-in-test sentence names only "one that places a path outside the hub, or one that leaves the board's HEAD unequal to its upstream tip". `os.MkdirTemp` falls under "a path outside the hub", but the other three calls fit neither case. A mover whose fixture makes one of them has a review that refuses the build and a D7 that never says the step stays in the test after the copy. Adding them to the stay-in-test sentence would close the gap.

Tool calls:
- One grep of the files that define the D7-listed fixtures, for `Setenv`, `MkdirTemp`, `TempDir()`, `--global`, `Chdir` and `Context()`. My context did not show those fixture bodies, and I needed to know whether a present build step reaches the refused calls. The hits were in test bodies, not clearly in fixture build steps, so they do not settle it either way.
