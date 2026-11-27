```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens. For each function and branch the diff adds or changes, run a concrete case and judge the result.

**List: function or branch, the case run, verdict**

- `runTest` loop, `internal/gatecli/run.go:173-232`:
  - Cases run: a green plan; a red tier; a halt on the last run, with `end.ran` and code 0; `--stop-on-fail` with plans left; a `--gate` key with lints.
  - With the report chain gone, nothing deferred or cancelled is left. The halt returns after `printReport` and `printFailedRuns` as before, lints still set code 1 only when code is 0, and `inHub` is still used. Correct.
- `flagShapeRefusal`, `internal/gatecli/run.go`: `--copies=-1` still refuses. The `--map` branch is gone and the flag no longer exists. Correct.
- `runInvocations` cap and guard start, `internal/gatecli/tierrun.go:619-646`:
  - Case run: a tier command with a marker and `samplePeriod` &gt; 0.
  - The cap is now `capFor(marker, period)`. `commandPID` and `request.commands` are gone along with their only readers. Correct.
- `processCap.run`, `internal/gatecli/processcap.go`:
  - Case run: list `[1,2,3,4]` at limit 3. It trips and kills all four, as at base when no commands were set.
  - Case run: a list error with `ErrUnsupported`. The cap still ends.
  - The only marked processes during a tier are the tier's own tree and the `--copies` siblings. The shared build runs before the tiers, so dropping `countable` counts no new process. Correct.
- `newProcessCap`: zero `sample` and `root` fields are no longer read. Correct.
- `memoryGuard.run`, `internal/gatecli/memoryguard.go:137-147`:
  - Case run: a tree with one runaway binary. `kills(testTrees(tree), killed)` kills it once, and the `killed` map stops a second kill.
  - `enclosing` was read only by `onKill`, which is gone. Correct.
- `memoryGuard.killTree`: with the `onKill` branch removed, it SIGKILLs `kill.tree.pids` as before. Correct.
- `memoryGuardApplies`: unchanged, and `run.go:161` still calls it. Correct.
- `verbDiffBase` moved to `internal/gatecli/redfirst.go:162`:
  - Cases run: empty `requested` with no parent; `requested` naming no commit; a merge-base read error.
  - Its body is byte-identical to the base copy in `mutate.go:116`. Correct.
- `noteScoper`, `internal/gatecli/redfirst.go:42`: for scoper `none`, the text now says "neither go nor dotnet, so red-first judged nothing". That is true at its one use, `redfirst.go:162`, which runs only for a scoper that is neither go nor dotnet. Correct.
- `goModuleOf`, `internal/gatecli/gomodule.go:135`:
  - Cases run: a config declaring only tier1; one declaring neither tier; a tier dir outside the worktree (`IsLocal` false).
  - It is the same body as the base `redundancyModule` with `moduleTiers` {0, 1}, and only the no-tier error text changed. Correct.
- `sourceTree`, `workingTree`, `commitTree`, `inModule`, `isModulePackage`, `modulePackageOf`, `goModule.relativePackage` and `testreachModule`:
  - Cases run: an absent dir (nil, nil); module rel "."; dir equal to the module rel ("."); a `testdata` segment.
  - All are verbatim moves of the base `redundancytests.go` and `reach.go` code. Each keeps a production caller: `redfirst.go:191`, `:199`, `:582`. Correct.
- `testreach.ChangedTests` and `declaredTests`, `internal/testreach/changedtests.go:64-149`:
  - Cases run: an identical test moved to another file (matched by name, so unchanged); the same name in two files of one package (matched per file); a comment edit inside the body (text differs, so changed); a doc-comment edit only (`fn.Pos()` starts at `func`, so unchanged, as at base); an unparsable file (error naming it); empty FileSets (nil result).
  - `isTestFunc` excludes receivers and `init`. So the base's `init` ordinal and `Type.Method` naming never applied to tests, and `funcKeyName` reduces to `fn.Name.Name`.
  - Comparing the text equals comparing its SHA-256 except under a collision. Correct.
- `testFileImportPath`: it inlines the base's `directoryImportPath`. Cases run: rel "", rel ".", a file outside the module, a `_x` segment. Correct.
- `testcover.Toolchain.goOutput`, `internal/testcover/runner.go:243`:
  - Case run: a failing `go test -c`. The error carries stderr.
  - No WaitDelay is set any more, so the dropped `ErrWaitDelay` branch cannot be hit. `Dir` was never set by the surviving caller, `cmd/testtiming/redundancy.go:314`. Correct.
- `testcover.Toolchain.testCommand`:
  - Case run: `TestTimeout` 0 gives an unbounded plain command, as at base when `Ctx` was nil.
  - Case run: `TestTimeout` 2s gives `WithTimeout(Background)` and a group kill, as at base.
  - Correct.
- `testcover.Toolchain.RunAlone`:
  - Case run: a failed run with `MemoryLimit` &gt; 0 and `killed` set gives MemoryCapped.
  - Case run: a failed run with no limit gives a plain fail.
  - Case run: a deadline gives TimedOut before capped.
  - With `GuardKilled` gone, there is no branch left reading a nil func. Correct.
- `testcover.Judge`, `internal/testcover/verdict.go:52`:
  - Cases run: empty runs (empty slice); C and D covering only each other (one removable, as in the base kept=nil path); a skipped run (Reason set, not counted).
  - It is the base `JudgeAgainst(runs, nil)` with `all == runs`, so `out[:len(runs)]` equals `out`. Correct.
- `SpawnVerdict` with its json tags dropped: no surviving code marshals it. `cmd/testtiming` keeps it only in in-memory structs (`redundancy.go:75`, `:90`). Correct.
- `gateslot.decodeConfig`: `fixer_timeout_min: 0` gives the same refusal text as the base loop. Correct.
- `gateslot.decodeGates`:
  - Case run: an entry `{tiers: [0], scope: diff, reports: [reach]}`. `yaml.Node.Decode` into GateKey, which has no `Reports` field, is non-strict, so the key is ignored.
  - Prune treats `gates` as a per-name map and does not descend into it (`yamlengine/layers.go:524`), so no warning and no refusal follow. Correct.
- `gateslot.Config.Gate`: with zero tiers it returns Scope and Lints. With tiers declared it refuses an undeclared tier unchanged. Correct.
- `gatefixer.rowWrap`, the `waitOnly`, `spawning` and `spawnAndJudgeAgain` paths, `internal/gatefixer/rowgate.go:159-226`:
  - Case run: a live record with `afterFixer`. It now calls `inner`, which equals the base `(*RerunMark)(nil).Run(inner)` whenever the entry carried no mark.
  - Every remaining resolver builds entries without a mark, so behaviour is unchanged. Correct.
- `shedrecipe.resolveGateEntry`, `internal/shedrecipe/entries_gate.go:165`: the `verify` entry is now the plain closure. This equals the base path with `VerifyReach` nil. Correct.
- `weaverengine.batchGateSpec`, `internal/weaverengine/strand.go:140`: the last, non-card entry is now the plain `batchGateClosure`. This equals the base path with `ReachPending` nil. Correct.
- `shedadapters.ReadRubric`:
  - Case run: the shipped code-review rubric, which no longer carries the three markers. It fills.
  - Case run: an untouched board copy of the old rubric that still carries the markers. `FillWith` refuses unfilled markers. But stencilstore's ModeProduction refreshes an untouched copy whose shipped default changed (`internal/stencilstore/doc.go:7-14`), and the hub copy at `_bolt/_lym/stencils/loom/loom-rubric-code-review.md` is stamped as shipped. Correct, given the existing refresh.
- `Bouncer.rubricNotes` and `reviewRubricNotes`: the dropped fields leave GateTiers, Amendment, ReviewInput and WebsterRecord as before. Correct.
- `loomCLI.wire`, `internal/loomcli/wiring.go:834`, `:925`: the removed seams were nil-legal, and `envkit`'s nilLegal list drops the same names. `VerifyHold` stays wired. Correct.

**Findings**

None. Every changed function and branch I ran gives the base result, minus the removed feature, and no error is swallowed or misreported. `go vet` over the touched packages exits 0 (manifest).

**Tool calls**

1. `git show` of the base `mutate.go`, `reach.go` and the other removed files, to compare `verbDiffBase`, `reachModuleOf` and `noteScoper` with the moved copies; the manifest held only the new side.
2. `git show` of the base `redundancy.go` and `redundancytests.go`, for the bodies of `redundancyModule`, the trees and `isModulePackage`; the moved helpers' originals were not in context.
3. A grep of `internal/gatecli` for the moved helpers' callers, and `run.go:160-235`, to confirm each helper keeps a production caller and the post-removal tier loop is consistent; the diff showed only the removed hunks.
4. `stencil.Fill` and a search of the hub for rubric copies still holding the removed markers, to judge whether `ReadRubric` fails on an old copy; this was one hop beyond the manifest.
5. The `stencilstore` doc and the hub rubric copy's header, to learn whether an untouched board copy is refreshed; this settled case 4.
