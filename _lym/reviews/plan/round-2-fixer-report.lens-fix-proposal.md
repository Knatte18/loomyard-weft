```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the F16 proposals the fixer carried out as written, in `00-overview.md`'s coverage-of-namespace-sites decision and card 7's four new `TestOneRepositorySecondMachine_AdoptPullAndReplay` bullets.

**MEDIUM** — card 7, bullet "live local branch".
- **Summary:** As written, the step can't be carried out in this test's sequence. It needs a local records branch with no pair, and the proposal never says how to build that state.
- **Evidence:**
  - Earlier in the same test, the "adopt" bullet has `Topology.Add` of the slug on the second machine create the pair. A second `Topology.Add` of that same slug would then be adding a pair that already exists.
  - `probeLiveLocalWeft` runs only when `weftBranchExists` is true and the pair is being newly added (`internal/fabricengine/add.go` around line 248, `weftBranchAlreadyExists`, then `weftLiveness`).
  - The two-repository test reaches that state in a different way, in `internal/fabricengine/add_remoteleftover_integration_test.go` lines 115-124. It calls `Topology.Remove` with no remote step, recreates the local records branch at the old tip with `git branch`, moves origin ahead, and only then calls `Topology.Add`.
  - The bullet should name that setup, or a fresh slug built the same way. Without it, an implementer following the bullet in order either hits an existing pair or never reaches `probeLiveLocalWeft`.

The other three bullets (checkout, records push, owner archives) and the overview edit hold up.
- **Checkout:** `targetWeftTip` is reached with the local branch absent, and an un-namespaced fetch of `refs/heads/&lt;b&gt;-weft` from the bare fails, so the row tells the two apart.
- **Records push:** "adds no ref under `refs/heads/`" catches an explicit identity refspec.
- **Owner archives:** a records-side read left un-namespaced would not see the tag under `refs/lym/tags/`, so the row fails.

Tool calls:
1. `grep`/`sed` on `internal/fabricengine/add.go` and `weftwiring.go`: to see how `Add` treats an existing local records branch. My context lacked `Add`'s body.
2. `grep` across `internal/fabricengine/*_test.go`: to find the existing tests of the fast-forward probe. My context held no test files.
3. `grep` across the `Add` tests: to find the test that builds a behind local records branch.
4. `sed` on `internal/fabricengine/add_remoteleftover_integration_test.go` lines 88-175: to read that test's setup, which my context lacked.
