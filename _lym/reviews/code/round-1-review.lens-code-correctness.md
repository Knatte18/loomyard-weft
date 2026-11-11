```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens over the diff, with one entry per changed function or branch.

**Per-function list**

- `weftConflictStop` (`pull.go:464`), input is the `ErrReplayConflict` case with `paths=outcome.Conflicts` (non-empty, `["notes/a.md"]`), `ahead=["abc123"]`. Correct.
  - Before the change the literal built `WeftStopError{Branch, Paths: outcome.Conflicts, WayForward: recordsConflictWayForward(f.weftPath, branch, outcome.Conflicts, ahead, push-verb), Err: err}`.
  - The extracted function builds the same four fields with the same arguments. `err` is passed as `cause`, so `Err` is unchanged.
  - The call site `pull.go:439` passes `(branch, outcome.Conflicts, ahead, err)`, which matches.
  - Behaviour is identical.
- `weftConflictStop`, boundary input of nil `paths` and empty `ahead`. Correct.
  - `recordsConflictWayForward` takes the `len(paths) == 0` branch and uses "settle any conflict".
  - `joinOrNone` renders the empty commit list as its none value.
  - Nothing panics.
  - `lymVerbDir` falls back to the root path if `lymcwd.NamedWorktree` fails and logs a warning. If `warpPath` is empty it returns "", so the push verb reads "in ". This is the same as the removed literal.
- `WeftConflictStopForTest` (`export_test.go:839`). Correct.
  - It builds a `Fabric` with only `warpPath` and `weftPath`. `weftConflictStop` reads only those two fields, plus `lymVerbDir` and `recordsConflictWayForward`, which are pure functions. There is no nil dereference.
  - It passes `ErrReplayConflict` as the cause.
  - The test asserts only `WayForward`, so the hardcoded cause is not asserted.
- New row `WeftConflictStopMergesAtTheRecordsRootAndPushesInTheAnchoredDir` (`failedpush_test.go:491`). Correct.
  - I walked the production format against the hand-built `want`, with `paths=["notes/a.md"]`, `commits=["abc123"]`, `branch="task-weft"`, `worktree=anchoredWeft`. The text is the same piece by piece:
    - `found` is "origin/task-weft holds commits that change the same paths as this branch's (paths: notes/a.md; local commits: abc123)".
    - Then comes "; to keep origin's side, run `git reset --keep origin/task-weft` in <weft>, which drops the local commits; ".
    - Then "to keep this machine's, run `git merge origin/task-weft` in <weft>, settle the named paths and commit, then run `lym fabric push` in <dir>, whose push is then a fast-forward and meets no replay".
  - `anchoredDir` is `anchoredTask/src/app`.
    - Lines 253-255 define it.
    - Line 259 writes the `src/app` marker in the weft.
    - `lymVerbDir(anchoredTask)` resolves through `NamedWorktree` to `AnchorPath()`, which I expect to be that path. This matches the existing `AnchoredCodeNotFastForward…` row.
  - The row would fail if `lymVerbDir` fell back to the root or if the merge directory moved.
- `newFailureFixtureAt` and `newFailureFixture` (`pushclasses_integration_test.go:38-48`). Correct. The old function delegates with `"."`, which is the previous `Shape{Anchor: "."}`. Other callers are unchanged.
- `newPairFixtureAt` and `newPairFixture` (`pushclasses_integration_test.go:119-128`). Correct. The old function delegates with `"."` and the body is otherwise unchanged. The body after line 128 was not in the diff, so I could not check that it makes no anchor-dependent assumption. Its use of `lymcwd.ResolveWorktree(PairCodeWorktree)` and `p.warpPath` is anchor-agnostic as far as the diff shows.
- Anchor-change row `OntoAnUpstreamCarryingAnotherMarkerSetsTheAnchorInTheAnchoredDir`, hub anchor `"backend"` (`pushclasses_integration_test.go:482-491`).
  - The expected string is `run \`lym fabric anchor <WorktreeName> --set sub\` in <warpPath>/backend, which integrates the other marker`.
  - This depends on the way forward for the anchor-change class in `failedpush.go` (lines ~297-335, not in my context), which sets the directory through `lymVerbDir(CodeWorktree)`.
  - The original row only asserted `"lym fabric anchor"`, so the new assertion is stricter. It cannot be confirmed from my context that the suffix ", which integrates the other marker" and the `WorktreeName` form match production. The verify gate ran on this tree, so I count it as passing. I could not confirm the match, and the other forks cover the test side.
  - Possible issue: `p.location.WorktreeName` is used as the slug, while the fixture's slug is `anchorstop%d`. `AddPair` presumably sets the two equal.
- The other two rows keep `"."`, so they still build the same fixture as before. Correct.
- Loop variable `i` is used inside `t.Run`. Go 1.22+ semantics make this safe. The slug differs per row.

**Findings:** none under this lens. The production extraction is behaviour-identical.

**Tool calls**
1. One grep of `failedpush_test.go` for the `anchoredTask`, `anchoredDir` and `anchoredWeft` definitions. My context did not hold where they are defined, and I needed to confirm they are in scope and anchored.
