```yaml
lens: code-affected-set
outcome: reported
```

**Task:** the code-affected-set lens, covering what the build-tag-to-run-time-selector change forces outside the lines it touches.

**Per change: set found, covered or not**
- **`LYM_TEST_TIER` selector (`tiergate`).**
  - Set: `tmuxkit`, `impactset.FileTier`, the `cmd/lym` tier, purity and LLM guards, `cmd/testtiming`, and `loomshed`'s `test-tier-unstated` way-forward text.
  - Covered, except where findings 1 to 4 apply.
- **Tier tags removed from test files.**
  - Set: the PATTERN entry, `pattern/PATTERN-test-speed.md`, `pattern/PATTERN-test-isolation.md`, `contracts/specs/loom-plan-spec.md`, `docs/benchmarks/running-tests.md`, and the doc comments and file headers that describe tags.
  - The PATTERN entry, both background files, the plan spec and the benchmarks doc are updated.
  - The doc comments and file headers are not all updated (findings 1, 2, 4).
- **`allow_tmux.go` and `allow_none.go` replaced by `tmuxallowed.go`.**
  - Set: `tmuxkit.go` `afterRun` and `checkSockets`, `PATTERN-test-isolation`.
  - `afterRun`, `checkSockets` and the PATTERN text are updated.
  - The package doc is partly stale (finding 1).
  - Nothing else references `allow_tmux` or the old `tmuxAllowed` variable.
- **13 `!tier1` `TestMain` companions removed.**
  - Set: each package's remaining `TestMain` and its header comment.
  - Behaviour: the remaining `TestMain` matches or supersedes the removed one in 12 of the 13 packages. The exception is battencli (finding 3).
  - Headers: the remaining files' headers are stale (finding 2).
- **`testtiming` sets `LYM_TEST_TIER` from `-tags`.**
  - Set: the `cmd/testtiming` package doc, `spawnscan.go`.
  - Covered, apart from finding 4.
- **`-tags` readers left as they are** (`crucible/`, `planparser/verifymodule.go`, `gatecli`): finding 5.
- **Declared PATTERN changes.**
  - The constraints section declares no changes list.
  - The `PATTERN-test-speed` entry and background file and the `PATTERN-test-isolation` background file are updated, so there is nothing missing.
- **Commit messages and `Pattern-Departure:` trailers.** No trailers are present, so there is nothing to judge.
- **Stencils.** The diff changes none.

**Findings**

1. **LOW, consistency.** `internal/testkit/tmuxkit/tmuxkit.go:10` and `:18` still describe tags. Line 10 says "every reed.yaml a tagged test renders", and line 18 says "no finding in a package with either tag". Line 20 was updated, so the doc now mixes both models.

2. **LOW, consistency.** Headers of the surviving `TestMain` files still describe tag-split binaries.
   - `internal/battencli/testmain_integration_test.go:1-6` says it wires "this package's integration test binary".
   - `internal/hubgeom/testmain_integration_test.go:1` says "wires the integration-tagged tests".
   - Also stale: `internal/loomcli/testmain_test.go:4-13`, `internal/websterengine/testmain_test.go:6` ("only the //go:build tier1 files ever reach git") and `internal/hubforge/testmain_test.go:10`.
   - These files now serve every tier. The `_integration_` file names remain, but they carry no tag.

3. **LOW, design.** Removing battencli's `!tier1` `TestMain` drops the tier0 stub that refused a pair push.
   - Removed (base `internal/battencli/testmain_test.go`): `spawnPairPush` returned the error "unit tests push no pair".
   - Remaining (`testmain_integration_test.go`): `spawnPairPush` calls `fabricengine.PushPairAnchored` for real.
   - Call sites: `wire.go:980` and `:1425`.
   - A tier0 test that reaches a pair push now runs git instead of failing loudly. The purity guard cannot see this because the call goes through a seam.
   - The same merge also gives tier0 runs `HermeticGitEnv` in hubgeom, landingshed, logger, preflightshed and shedcli, and `FABRIC_SKIP_PUSH=1` in hubreconcile. Those are benign.

4. **LOW, consistency.** `cmd/testtiming` docs still describe tagged files.
   - `cmd/testtiming/main.go:51` says "A test in a file with no tier tag is always judged, and a tier-tagged test with a call the scan cannot resolve is excluded".
   - `spawnscan.go` now classifies by `FileTier`, and a `tier0` file has no tag.
   - Smoke-test headers also still say tag-run, for example `internal/reedcli/smoke_test.go:6`, `internal/shuttlecli/smoke_run_test.go:8` and `internal/burlerengine/smoke_round_test.go:11`.

5. **MEDIUM, consistency.** `-tags tierN` still drives things that no longer select a tier, so silent skips are left behind. The summary says it left these as they are. Before the orch change lands, the gate sets the variable, so the gate's own runs are correct. The consumers below are not.
   - `crucible/orchestrator-prompt.md:119` and `:122` still run `go test -tags tier2 … -run Smoke` (and `-tags tier3` for LLM-driving modules). With `LYM_TEST_TIER` unset these select `tier0`, so every smoke test is skipped and the run goes green. This is the failure mode the "no test may go silent" decision (creel #369) targets.
   - `internal/planparser/verifymodule.go:145` and `internal/planparser/validate_test.go:2071-2081` treat `-tags tier2` and `-tags tier3` as meaningful.
   - `contracts/specs/loom-plan-spec.md` line 340 still says "Untagged tests are fast by construction" in its tier text.
   - `internal/gatecli/doc.go:126` and `:285-293` document `-tags tierN` commands. These describe the hub config and are correct until the orch drops the tags, so I flag none of them.
   - `docs/benchmarks/test-suite-prune-rest.md` is a measurement report and is exempt.

**Tool calls**
1. Grep for stale tag wording across `.go`, `.md` and `.yaml` outside `_lym`, `.lym` and `docs/research`. My context held only the diff, not the untouched files.
2. Diff of `pattern`, `docs`, `contracts` and `loomshed`, filtered to changed lines. The earlier diff output had been truncated before the `pattern/` hunks.
3. Diff of `pattern/` and `PATTERN.md`, plus a grep for `allow_tmux`, `tmuxAllowed` and `-tags` consumers. This was to find the remaining readers of the old model.
