```yaml
lens: code-affected-set
outcome: reported
```

Task: affected-set lens. For each change the diff makes, find what reads, calls or describes it outside the touched lines.

**List: per change, the set found, covered or not**

1. **`withTimeoutMin` replaces `context.WithTimeout`** (`tierrun.go`, `build.go`).
   - Set: `runTierCommands`, `runBuild`, `runInvocations` and `prebuildLym` under `tierCtx`, the `doc.go` "Timeouts" section, and the `copies.go:66` prebuild of a copies call.
   - The tier, the build and the doc are covered.
   - `copies.go:66` is **not covered**: it still uses a monotonic `context.WithTimeout` over the same `timeout_min`.
   - The other timeout sites (`redundancy.go:523`, `redundantmutants.go:522`, `run.go:66`) bound other keys and are outside this set.
2. **The child marker and `killMarkedLeftovers`** (`child.go`).
   - Set: the callers `build.go:54` and `tierrun.go:567`, `proc.PIDsWithEnv` (Linux and Windows), its other user `processcap.go:53`, the `doc.go` "Child lifetime" section, `proc/doc.go:24-26`, and PATTERN-child-processes.
   - All covered.
   - The direct SIGKILL follows the precedent of `killLeftovers` and `processCap.kill`; a tier child is no runproc-registered run process.
   - `withBuildServerOff` (`run.go:920`) keeps a gate's own .NET build servers from lingering, so no shared server carries the marker.
3. **The vstest counter's multi-line summary** (`counters.go`).
   - Set: `observeFailure`, `startInvocation`, `replaceInvocation`, the `doc.go` "Counters" section and `TestCounters`.
   - All covered.
4. **A caller's `--filter` merged into `{filter}`** (`tiers.go`).
   - Set: `renderTierCommand`, `filterArgument`'s caller `TestFilterArgument`, `run.args` consumers (`green.go:344/385/448`, `resultsArgsRefusal`, the watch-skip check in `isFilterFlag`), and red-first dotnet, which passes terms rather than args.
   - Also the help text (`cli.go`), `doc.go:78-83` and the `tierFilterLeaf` constant.
   - All covered, except the list of planning refusals (finding 2).
5. **The no-test notice and `otherTiers`** (`tierrun.go`, `tiers.go`).
   - Set: `noTestNotice`, both return paths of `planTierRuns`, `notRunTiers`, and `doc.go:348-352`.
   - All covered.
6. **The tier0 refusals** (`gatecli/tiers.go:95`, `gateslot/tiers.go:388`, `:415`).
   - Set: the tests that assert them by substring (`gateslot/config_test.go:883,918`, `test_integration_test.go:2178`, `tiers_test.go:506,523`), which still match, and `doc.go:165`.
   - Callers of `Config.Tier` (`scope.go:190,544`, `dotnetscope.go:70` and others) wrap the error and stay correct.
   - The way the key path is built is **not covered** (finding 1).
7. **The Burler fixer stencil paragraph.**
   - Set: `burlerengine/fix.go` reads the stencil, and its "fixer-report's deferred section" exists (`burler-step-fix.md:48,88`).
   - The paragraph names no denied step and no engine term, and stencilstore keeps no hash registry that would need updating.
   - Covered.
8. **The help text cut** (`cli.go`).
   - The dropped clauses are still stated in `doc.go:49` (the copies lease) and `doc.go:172-178` (inert-only scopes).
   - Covered.
9. **Commit messages and declared PATTERN changes.**
   - Every fix commit carries a `Root-Cause:` trailer, and no commit carries a `Pattern-Departure:` trailer.
   - The directive declares no PATTERN changes.
   - Nothing to raise.

**Findings**

1. **BLOCKING, class `consistency`, at `internal/gatecli/tiers.go:95`, `internal/gateslot/tiers.go:388` and `internal/gateslot/tiers.go:415`.**
   - The new refusals build the key name as `keyName("tiers."+tierName(0))` and `keyName("tiers."+tierKey(n))`, from a repeated `"tiers."` literal rather than a named constant.
   - PATTERN-config-key-names requires a key name rendered "from named constants", and PATTERN-code-conventions requires a name that can go stale to be one named constant.
   - `gatecli` already has `tierKeyName` (`keynames.go:38-40`), which also holds the literal.
   - Older sites in `gateslot` use the same form (`runlog.go:40`, `config.go:400,493`, `tiers.go:187,191,205`), but the rule outranks that precedent and the diff adds three more.
   - Evidence: `tiers.go:95`, `gateslot/tiers.go:388,415`, `gatecli/keynames.go:39`.
2. **LOW, class `consistency`, at `internal/gatecli/tiers.go` (`planTierRuns` doc comment, about lines 85-92) and `internal/gatecli/doc.go:163-166`.**
   - `planTierRuns` now refuses a caller `--filter` for a command with its own `--filter` and no `{filter}`.
   - Neither the function's doc comment, which lists its refusals, nor the package doc's list of planning refusals names it; only the `{filter}` passage at `doc.go:361-362` mentions it.
   - Evidence: `tiers.go:122-125`, `doc.go:163-166`.
3. **LOW, class `consistency`, at `internal/gatecli/copies.go:66`.**
   - A `--copies` call's shared prebuild is still bounded by `context.WithTimeout` on the monotonic clock.
   - The updated `doc.go` "Timeouts" section says the tier's `timeout_min` "bounds the prebuild" and "counts wall-clock time as well as awake time", which is false on this path.
   - Evidence: `copies.go:66`, `doc.go:316-320`.

Tool calls:
1. Searched for the tier0 refusal texts, red-first's filter use, stencils naming a deferred section and contract docs naming `--filter`: their consumers were outside the manifest.
2. Searched for `"tiers."` key-path literals and the key constants, and looked for a stencil hash registry: needed to judge the key-name constraint and the stencil's affected set.
3. Listed `internal/stencilstore` and grepped the fixer stencils for "deferred": needed to check that no pin needs updating and that the deferred section exists.
4. Read `stencilstore/pins.go` and `doc.go`'s planning-refusal list: needed to confirm no hash registry and to check the refusal enumeration.
