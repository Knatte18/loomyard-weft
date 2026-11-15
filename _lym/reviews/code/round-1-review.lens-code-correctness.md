```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran each changed function and branch on concrete inputs. One LOW finding; everything else holds.

**Functions and branches, the case run, and the verdict**

- **`cmd/lym/main.go` `main` → `run(os.Args[1:], os.Stdout, os.Stderr)`** (main.go:55-61): correct.
  - Case: `os.Args = ["lym"]`. `os.Args[1:]` is an empty, non-nil slice, so `SetArgs([])` runs and the root shows its help. Before, `args == nil` made cobra read `os.Args[1:]`, which gives the same empty list.
  - Case: `lym board bogus`. The JSON envelope still goes to stdout via `RunRoot(root, stdout)`, and cobra's text goes to stderr, so the streams stay split.
  - `notifyExitAndSweep` runs once, then `lymtemp.Remove` and `os.Exit(code)`. Nothing else reads `os.Args`.
- **`run` in tests** (one buffer passed as both writers): correct. stdout, stderr and the envelope all land in the buffer, as before.
- **`configcli.resolveReal`'s `realSync`** (configcli.go:806-816): correct.
  - Case: ctx carries `"/hub/feat"`, so `ExecuteIn(fabriccli.Command(), "/hub/feat", w, ["sync"])` runs. That is the same call `RunCLIIn`'s non-empty branch made.
  - Case: ctx carries no cwd. `CwdFrom` falls back to `Getwd` (cwdcontext.go:36-41), which either returns a non-empty dir or an error, and the error returns before `realSync` is built. So the empty-cwd panic in `ExecuteIn` cannot be reached.
- **reedcli `executeReedIn`** (smoke_test.go:149-156): correct. `""` goes to `Execute` and `"/x"` goes to `ExecuteIn`, which is `RunCLIIn`'s behaviour. The empty-cwd callers (smoke_test.go:146, 252) go through it.
- **reedcli direct `ExecuteIn` helpers** (`paneIDForStrandIn`, `statusStrandLive`, `assertDiagnosticVerbsRefuseAndDepositNothing`): correct. A grep found no caller passing `""`, so none can hit the empty-cwd panic.
- **Sampled `RunCLIIn(dir, …)` → `ExecuteIn(Command(), dir, …)` swaps** in battencli, loomcli and gatecli tests: correct. Each passes the same arguments, including `nil` args.
- **`hubforge.NewRecordsStopError`** (recordsstop.go:7-12): correct. It builds the same `&WeftStopError{Branch, Err, WayForward}`, and `RecordsStopMessage`'s `errors.As` still matches it.
- **`reedengine.tmuxProcessName`, moved into the Windows file** (proctree_windows.go:66-77): correct. The body is unchanged. Inputs `C:\tools\PSMUX.EXE` → `PSMUX.EXE` and `/usr/bin/tmux` → `tmux.exe`. Its caller is proctree_windows.go:87.
- **Moved bodies:** `CommitWeftForTest`, `SnapshotWarpSHAForTest`, `PushAnchored`, `WireJunctions`, `CodeLymLink`, creel `Post`/`PostWait`/`PostInterrupt`, `Board.Promote`/`Store.Promote`, `IsDraftHold`, codexengine `New`, `TemplateConfig`, `reflowSource`, `RoleTime`, `archivedPlanWebsterDirs`, preflight `Wired`, preflightshed `NewPreflight`/`newPreflightWith`, and planglyph `ValidateFormat`/`ValidateRework`/`Validate`: correct. Each is byte-equivalent to the deleted production body, so behaviour is the same.
  - Case: `SkipGit` → `("", false, nil)` with no lock taken.
  - Case: the lock acquisition fails → the error is wrapped as before.
- **`discussionparser.AppendDecisions`:** correct. Only the doc changed. Empty `ds` → error.
- **`KindCommitsDropped` doc rewording** (mutation.go:97): correct. Live writers remain at replay.go:791 (Bolt replay) and destroy.go:2018, so the "commits dropped from a checkout" wording holds.
- **`pattern` package after `Check` moved:** correct. `continues`, `EntryRuleText` and `KindOverviewHeadroom` are unchanged.
- **Guard `functionName`:** correct.
  - `func (r *T[K]) M()` → `StarExpr` → `IndexExpr` → `"T.M"`.
  - `func F()` → `"F"`.
  - A receiver that is not an ident falls back to the method name.
- **Guard `isMarkerMethod`:** correct. `func (x) isPhase() {}` → true. An exported or non-empty method → false.
- **Guard `inTestSupportPackage`:** correct.
  - `internal/testkit/scankit` → true.
  - `internal/testkitx` → false, because the check requires a `/` after the key.
- **Guard reachability keyed on `token.Position`:** correct.
  - `init` functions: `init#N` keeps the declaration's name position.
  - Instantiated generics share their origin's position.
  - Promoted-method wrappers reach the embedded method.
- **Guard `packageFunctions` roots:** **LOW** (see the finding below).
  - Case: a generic `testkit` helper `func Fixture[T any]()` that only `_test.go` files instantiate. Test files are not loaded, so no instance exists.
  - The guard skips the generic origin as a root, so a production function only that helper calls reads as unreachable. That is a spurious failure the stated rule does not produce.

**Findings**

1. **LOW** · class `design` · `cmd/lym/testonlycode_linux_test.go:184-206` (`packageFunctions`)
   - Summary: test-support roots leave out generic functions and the methods of generic types, so the guard would wrongly flag a production function only a generic kit helper calls.
   - Evidence: `packageFunctions` lines 189 and 194 skip anything with type parameters. The guard loads no test files (`packages.Config` at lines 115-118 sets no `Tests`), so a kit generic that only tests instantiate has no instance in the program. The pattern says every function of a test-support package is a root (`pattern/PATTERN-test-only-code.md:26`), so this is a gap against its own rule.
   - The current tree passes; the gap is latent.

Tool calls:
1. Read `cmd/lym/main.go` lines 40-100 and grepped for `os.Args`: the manifest had only the `run`/`main` diff hunk, not the surrounding `main` body (exit and cleanup) or the other `os.Args` readers.
2. Read the reedcli `smoke_staterecovery_test.go` helpers and grepped for empty-cwd helper calls: needed the callers one hop past the manifest's `smoke_test.go` lines, to rule out an `ExecuteIn("")` panic.
3. Grepped the reedcli helpers again for `""` callers and diffed the battencli/loomcli/gatecli test replacements: to confirm the argument pass-through for swaps the manifest did not show.
4. Read `configcli.resolveReal` in full, `lymcwd.CwdFrom` and the `KindCommitsDropped` writers: needed to know where `cwd` comes from (whether it can be empty) and whether `commits_dropped` still has a live writer.
