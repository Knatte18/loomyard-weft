```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each Decision end to end on one concrete case, feeding every step what the step before it actually produces.

**Per-decision list**
- D1, a function in `internal/gatecli` reached by an own test and by a `cmd/lym` test: purpose met. Own-package reach goes through `BlockKey.File`/`Func` (`internal/testcover/blockkey.go:28-37`) and cross-package reach through `Map.Funcs`.
- D2, a new function plus one function with a changed body, merge base against worktree: purpose met.
- D3, the Darn-Write post-session verify on a clean tree, `darn-write` key listing `reach`: **purpose not met**. The writer never sees the note (F1).
- D3, a refresh of a clean package that links a dirty package: **input differs**. The measurement writes no map (F3).
- D3, the webster fork's `batch` gate, run by the fork itself in the foreground: purpose met. The fork reads the call's stderr (`internal/websterengine/doc.go:124`).
- D4, a function reached by more than five tests: met.
- D5, Code-Review after Darn-Write at the same HEAD: met.
- D6, a function the maps don't hold: met.
- D7, a red report near the 80-line cap, read by loomshed's `namedFailures`: **broken**. The added lines can cut the report (F2).
- D7, the card fixer reading stderr through `fixAndRerun`: met.
- D8, a parent-branch walk: met in mechanism, with a loose D2 hand-off (F5).
- D9, a fresh worktree map and a changed `if` line: met. `Mutant.Line` and `Mutant.Blocks` hold rendered `BlockKey`s, matchable against `Map.Blocks`.
- D10, a `darn-write` key listing `reach, mutation`: met, provided D3's refresh ran.
- D11, `lym gate mutate` run in the prime: **purpose not met**. Every package is not judged (F4).
- D12, the hub base `darn-write`/`batch` entries without `reports`: met. Config load fills the template's `reports` (`internal/gateslot/doc.go:113-114`; hub `gate.yaml:44-51` sets none).

**Findings**

F1. BLOCKING, design, `_lym/discussion/decision-record.md` D3 (lines 75, 79-80).
The Darn-Write and Code-Burler-Fix writers never read the reach note, so D3's stated purpose fails for both keys.
- Those gates run post-session through `verifytree.Verify` in loomshed.
- A pass returns `GateResult{Passed: true}` with no findings, so the note printed to stderr stays in the verify log and no writer reads it.
- A `StatusSkipped` tree runs no step at all.
- The record names exactly these gates as the writer's own gate.
- Evidence: `internal/loomshed/verifygate.go:183-188`; `internal/verifytree/verify.go:300-303`.

F2. MEDIUM, design, D7 (lines 128-130) and its acceptance criterion at line 252.
Triage lines can turn a parseable report into a cut one, which `gateslot.FailedTests` refuses.
- The failure report has one shared cap: `failureReportLines = 80`. A report that overruns it ends in a `... N more lines cut` line.
- `FailedTests` returns `ok=false` on a cut report, because that line falls to `reportRunTier`, which fails.
- A report holding fewer than 80 lines before triage but more after is therefore no longer parsed.
- That drops loomshed's `runFailedTestsAlone` rerun and landingshed's named-test path, which contradicts the "same failed tests" criterion.
- Evidence: `internal/gatecli/failurereport.go:29-30,200-215`; `internal/gateslot/runlog.go:290-293,333-337`; `internal/loomshed/verifygate.go:170-172,248`.

F3. MEDIUM, design, D3 (line 70) and Open risks (line 236).
D3's refresh check looks only at the package's own files, but the `--map` write also refuses when a linked package has a modified or untracked file.
- A clean changed package that links a dirty one is still refreshed.
- The refresh spends `reach_budget_min` running every test alone, then writes nothing.
- The note gives no reason for that package, or the wrong one.
- Evidence: `pattern/PATTERN-test-speed.md:305-307`; `internal/gatecli/testmap.go:585`.

F4. LOW, design, D11 (line 192) together with D3 (line 69) and D9 (line 159).
`lym gate mutate` run in the prime judges nothing.
- D3 refreshes nothing in the prime.
- D9 runs only on a refreshed map or a matching worktree map, and the prime keeps no anchor maps; its `--map` writes the shared ones.
- So every changed package comes back not judged, and the record states no bound or refusal for that case.
- Evidence: `internal/testcover/coveragemap.go:24-29` (Dir doc); `pattern/PATTERN-test-speed.md:221-222`.

F5. LOW, consistency, D8 (line 142) against D2 (lines 55-60).
D8 reuses D2 "between the commit and its first parent", but D2 is defined as base text against the worktree's files, with only the base side handed in.
- For D8 neither side is the worktree.
- D2's input contract doesn't cover two commits, so the walk's step input differs from what D2 takes.
- Evidence: the decision record's lines 55-60 and 142.

**Tool calls**
1. `internal/testcover/blockkey.go:27-38` and `internal/loomshed/verifygate.go:160-250`: the `BlockKey` fields and the Darn-Write verify flow were not in my context.
2. Grep for where the `batch` key is run: I needed to know who reads the batch gate's stderr.
3. `internal/websterengine/doc.go:115-135`: to confirm the webster fork runs `--gate batch` itself.
