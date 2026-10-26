```yaml
lens: code-tests
outcome: reported
```

**Task: the code-tests lens. Below is each behaviour the diff adds or changes, the test that pins it (or none), then the findings.**

## Behaviour → pinning test

**Engine: `FixPlanSpawnOrAttach`** (`internal/websterengine/planfixer_test.go`)
- No conflict record refuses with a way forward: `TestFixPlanSpawnOrAttach` refusal row "no plan-conflict record".
- Equal findings to an earlier fixer refuse, order ignored: row "an earlier fixer was given the same findings" (reversed order).
- Two fixers for the batch refuse: row "the batch already had two fixers".
- No master model-spec and an engine that cannot audit: two refusal rows.
- Spawn spec role, model, output file and skills; the prompt's dynamic values; the record's fields; stale reply removed; re-entrant attach to a non-terminal fixer: the "spawns at the master role…" subtest.
- A fixer from another batch does not count against this one: the same subtest, with a batch-1 earlier record.
- Spec `Timeout` and `Segment`: **none**.
- `HeadSHA` or `DirtyPaths` failing before the spawn: **none**.
- `Starter.Start` failing: **none**.
- `FindRun` failing, which stops the orphaned strand: **none**.
- `ReadConflictRecord` read error: **none**.
- An earlier terminal fixer for the same batch with different findings, count 1, still spawns: **none**. The off-by-one boundary of `planFixersPerBatch` is pinned only from the refusing side.

**Engine: `FixPlanAwait`** (`TestFixPlanAwait`)
- Verdicts `fixed`, `plan_writer` and a reply with no verdict line: three rows.
- HEAD moved overrules to `plan_writer` with a warning: one row.
- A new uncommitted path overrules: one row.
- A dirty path that was already dirty at spawn does not overrule: the "fixed reply" row, since `dirty-before.txt` is in both snapshots.
- Dead/died: one row.
- Dead/asking and dead/timeout for the plan fixer: **none**. They are shared `awaitColdFixer` code, and I saw no fix-verify rows for them either (not checked).
- `codeTouchedSince` read errors after the fixer: **none**.
- Warning on a failed stop, reply read or run-dir removal: **none**.

**Engine: `RecordPlanFixerTerminal`**
- Records status, reason and verdict, and reports false for an absent strand: `TestFixPlanAwait` and its trailing assertion.

**Engine: `ReadConflictRecord`**
- Findings and batch: indirectly, through the spawn subtest's `rec.Batch` and `rec.Findings` assertions.
- Absent record reads as no findings: the no-record row.
- `Cause` parsing: **none**. Nothing in the diff reads `ConflictRecord.Cause` either.

**Shared cold-fixer wait, `awaitColdFixer` and the `FixVerifyAwait` refactor**
- Pinned by the existing `verifyfixer_test.go` and `fixverify_test.go`, unchanged by the diff. I saw no change to the `fix-verify:` warning labels.

**State and lifecycle**
- Run respawn stops a non-terminal plan fixer and marks it dead/died, leaving a terminal one alone: `TestRun_EntryHousekeeping` in `runlevel_test.go`.
- Reset stops the plan fixers: `TestRemoveRecoveryStrands` in `stopper_test.go`.
- `masterAwaitedShellPrefixes` includes `fix-plan`: `TestRun_MasterSpawn` in `runlevel_test.go`.
- `PlanFixers` loads absent as nil (PATTERN-tolerant-records, acceptance criterion): **none** as a dedicated check. `fixplan_test.go` only loads a state that has a record.
- `recordedSessions` and `loadRunWrites` leave out plan-fixer sessions (D7): **none**. The existing subtest in `runwrites_test.go:63` seeds only `VerifyFixers`, so a regression that listed plan fixers would pass.

**CLI: `fix-plan`** (`internal/webstercli/fixplan_test.go`)
- Spawn call returns a running snapshot: the first call of every `TestFixPlanCmd` row.
- Attach and done/fixed envelope, with the verdict saved to state: row "a fixed reply…".
- Dead envelope, non-zero exit, stuck way forward, record saved dead: row "a gone strand…".
- No-record refusal and no spawn: the "no plan-conflict record" subtest.
- A `plan_writer` verdict or an overrule warning through the envelope: **none** at the CLI. The engine tests cover the verdict.
- Refusal under a covering pause: **none**. `batchpause_test.go:143` has a `fix-verify` case but the diff adds no `fix-plan` case, although `cli.go` and `fixplan.go` call `c.refuseCovered("fix-plan", …)`.
- Standalone `reedUp` failure, `SaveState` failure that stops the strand, and `fabricSync` failures: **none**.
- Lease released while the wait runs: **none**.

**Stencils and render**
- Prompt carries the record path, reply path, specs path and parent name: the spawn subtest.
- Required and optional markers for `webster-template-plan-fix`: **none**. `contracts/stencils/markers_test.go:50` lists the marker set for `webster-prefix-verify-fix-cold` and the diff adds no row for the new stencil.
- Friction note rendering per stencil: **none**. The per-stencil table at `internal/websterengine/template_test.go:962-968` lists master and verify-fix-cold and gains no plan-fix row.
- Master's plan-conflict rung: exempt, as static prose.

## Findings

1. **LOW, design** — a new assertion on a stencil's static prose. `planfixer_test.go`, spawn subtest, the `want` list. `"lyx webster rebaseline --card NN"`, `"lyx webster restore-plan"`, `"verdict: fixed"` and `"verdict: plan_writer"` are fixed text in `webster-template-plan-fix.md`. The dynamic values in that list (record path, reply path, specs path, `ly:orch`) are fine.

2. **MEDIUM, design** — D7's "not in the run audit's session list" is uncovered. No test shows that `recordedSessions` or `loadRunWrites` leave plan-fixer sessions out, and `runwrites_test.go:63` seeds only `VerifyFixers`. A regression that listed them would turn the fixer's legitimate plan-directory write into a correctness violation. The fix is one row: seed `PlanFixers` with a session and assert it is never audited.

3. **LOW, design** — the criterion "the new `state.json` field loads absent as nil" has no pinning test. The `fixplan_test.go` load only sees a state that has a record.

4. **LOW, design** — PATTERN-refusal-way-forward: several new refusals have no test that reaches them. These are `HeadSHA` and `DirtyPaths` failing before the spawn, `Starter.Start` failing, `FindRun` failing (which also stops the strand), `ReadConflictRecord` failing, and `codeTouchedSince` read errors. The covering-pause refusal of `fix-plan` is also unreached: `batchpause_test.go:143` has a `fix-verify` case but not a `fix-plan` one. One table-driven row or case each would satisfy the pattern.

5. **LOW, design** — the stencil registration tests were not extended. `contracts/stencils/markers_test.go:50` and the `internal/websterengine/template_test.go:962` per-stencil table each lack a plan-fix row. This matters most for the declared required and optional markers, which the stencil header describes.

6. **LOW, consistency** — `ConflictRecord.Cause` is parsed and never read or asserted anywhere. This is untested dead behaviour; drop the field or cover it.

7. **NIT, consistency** — the "no plan-conflict record" case in `fixplan_test.go` re-covers the engine's no-record row. It does add the CLI exit code and envelope, so it is arguably a distinct boundary. `TestFixPlanCmd` uses `t.Setenv` without a comment saying why it is not parallel, which `scribe:golang-testing` asks for. The existing `fixverify_test.go` may follow the same pattern; I did not check.

## Tool calls
1. `grep -rln "VerifyFixers|fix-verify|FixVerify"` over tests, to find the sibling tests that pin the existing fixer behaviour. The manifest did not list them.
2. `grep -rn "VerifyFixers"`, `refuseCovered` and `fixVerifyCmd` in tests, plus the stencil and marker names, to see which existing rows the diff should have extended.
3. `sed` of `batchpause_test.go:125-175`, `runwrites_test.go:60-95` and the `grep` of `markers_test.go` and `template_test.go`, to confirm that those tables have only fix-verify and verify-fix-cold rows.
