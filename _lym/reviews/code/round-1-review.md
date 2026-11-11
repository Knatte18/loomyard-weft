---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    origin: handler
    location: internal/gatecli/tiers.go:144
    summary: "The new explicit-targets refusal omits the role clause that doc.go says every config-gate refusal closes with"
    fix: "In planTierRuns at tiers.go:144 append roleClause to the fmt.Errorf format, as configRefusal does, so the message ends with the clause for a session that is not the prime. The existing refusal already names the prime step, so drop its trailing only-the-prime sentence and let roleClause carry it. Update the exact string in the tiers_test.go row for explicit targets that no listed tier's command takes to end with roleClause."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/run.go:222
    summary: "Nothing tests that runTest prints the unreached-project lines under the dotnet scoper and still exits 0"
    fix: "Add a row to TestGateTest_Scenario, or a runTest-level test, that runs a dotnet --card scope whose tier0 command holds no target placeholder while the card selects a project that command does not name. Assert the stderr line from unrunProjectLines and an exit code of 0. Do not add a second unit test of unrunProjectLines."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/dotnetscope.go:184
    summary: "namesProject counts a command that names only a built assembly such as Fast.dll as reaching every project, so the reported setup can stay silent"
    fix: "In namesProject treat a word with a .dll or .exe extension as naming a specific target, setting namesAny to true without matching the project, so only a command with no path-like words reaches every project. Add a row to TestUnrunProjectLines with a tier0 command naming Fast/bin/Debug/Fast.dll that expects the line for A/A.csproj."
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/dotnetscope.go:184
    summary: "namesProject matches by file name only, so a same-named project in another directory counts as reached"
    fix: "In namesProject compare the normalised relative path of the command word with projectPath when the word holds a directory part, and fall back to the base name only for a bare file name. Add a TestUnrunProjectLines row with A/Tests.csproj selected and B/Tests.csproj named by the command."
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: internal/gatecli/tiers.go:146
    summary: "The run-whole notice prints at plan time, so a later guard-step refusal leaves a stale notice before the error"
    fix: "In planTierRuns print the run-whole notices only once the plan can no longer be refused, by moving the loop at tiers.go:146 into a small helper called before each successful return of runs. Keep the notice text unchanged so the wantStderr rows still hold."
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/dotnetscope_test.go:36
    summary: "TestUnrunProjectLines has no row for a run with no invocation, a project selected twice, or a backslash project path"
    fix: "Add rows to TestUnrunProjectLines for a tier1 that takes targets but has an empty invocation list and so reaches nothing, for the same project selected in two tier shares yielding one line, and for a selected project path written with backslashes."
---

### [MEDIUM] Explicit-targets refusal lacks the role clause

**Location:** internal/gatecli/tiers.go:144
**Issue:** `doc.go` lists the new refusal with the others `planTierRuns` raises and says every such refusal closes with a clause for a session that is not the prime. `configRefusal` and the other `lym config gate` refusals append `roleClause`. The new `fmt.Errorf` does not, and its `exact` test row pins the message without it, so a non-prime session gets no instruction to report status FAILED.

### [MEDIUM] The dotnet warning wiring in runTest is untested

**Location:** internal/gatecli/run.go:222
**Issue:** The second ask of the board entry is a warning for test projects no tier reaches. `TestUnrunProjectLines` calls `unrunProjectLines` directly. The `cfg.Scoper == "dotnet"` guard and the `printLine` loop in `runTest` are covered by no test, so dropping or misplacing them would fail nothing.

### [MEDIUM] A command naming only a built assembly is treated as reaching every project

**Location:** internal/gatecli/dotnetscope.go:184
**Issue:** `namesProject` returns true when the words name no project or solution file. A tier0 command that runs `dotnet test Fast/bin/Debug/Fast.dll` names none, so every selected project counts as reached and the warning stays silent. The board entry describes tier0 as running only the fast test assembly, so this is a plausible form of the reported setup.

### [LOW] File-name-only matching confuses same-named projects

**Location:** internal/gatecli/dotnetscope.go:184
**Issue:** `namesProject` compares `path.Base` values. With `A/Tests.csproj` selected and `B/Tests.csproj` named, the pair counts as reached and no warning appears.

### [LOW] Plan-time notice precedes a later refusal

**Location:** internal/gatecli/tiers.go:146
**Issue:** The notice moved from `runListed` into `planTierRuns`, before the guard and vet branches that can still refuse. A refused call then prints a run-whole line before its error.

### [LOW] Unit rows missing for three branches of the new helpers

**Location:** internal/gatecli/dotnetscope_test.go:36
**Issue:** The zero-invocation return of `runReachesProject`, the `slices.Compact` merge of a project selected by two shares, and backslash project paths have no row.

## Rejected

- Lens code-tests F3 (a refusal row with two whole tiers): the multi-tier text is `tierNames` reuse, a helper that other tests already pin, so the row would add no behaviour.
- Lens code-tests F4 (an integration subtest for the refusal envelope): `runTest` passes the planner error straight to `output.Err`, a path older tests already cover, so a process-level row duplicates the planner row.
- Handler check of the focus questions on the other routes (`--card`, batch gate, `--gate` with a card): these warn through the run-whole notice and, under dotnet, the unreached-project line. The board entry says refuse or warn, so they are not defects. A plain tier run with no targets and a placeholder-free command still passes.
- Lens code-affected-set returned no findings.
