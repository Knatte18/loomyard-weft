```yaml
lens: code-affected-set
outcome: reported
```

Restating my task: I own the code-affected-set lens. For each change, I listed what reads, asserts or describes it, and whether the diff leaves any of that wrong.

**Per change: the affected set and whether the diff covers it**

1. **vstest `{args}` now opens with `--results-directory <dir> --logger trx;LogFilePrefix=gate --blame`** (`runInvocations`, `vstestResultsArgs`).
   - `internal/gatecli/doc.go` and the `runInvocations` and `vstestCounter` doc comments: covered.
   - `docs/dotnet-hub.md` "Tiers": it says a command holds `{targets}`, `{filter}` and "optionally `{args}`", and never mentions that the gate now adds its own words there. Not covered.
   - `docs/dotnet-hub.md` "A timing tier": its example `dotnet run ... -- {args}` with `counter: vstest` now gets `dotnet test` result arguments handed to a BenchmarkDotNet program. Not covered (finding 1).
   - The `docs/dotnet-hub.md` passage on Microsoft.Testing.Platform does not mention that such a tier, if it holds `{args}`, now receives `--logger trx`/`--blame`. The summary names this risk; the doc does not. Not covered (part of finding 1).
   - `record.Command` now carries a temp path that differs on every run. No production reader of the run record's `Command` treats it as an identity: green carry and the detached runner's join key on other things. Covered.
2. **The TRX results replace the console counts and failures; run messages become causes** (`replaceInvocation`, `resultsFileLines`, `runTierCommands`).
   - Readers of the counts and failures: `noTestNotice` subtracts `discoveredBefore`, which is taken after `startInvocation`, so it stays consistent. `runnerCrash` and `retryTier` see the same counts.
   - Copies and reruns build a new counter each.
   - `doc.go` "Failure report" text: covered.
3. **`Config.Gate`'s unknown-key refusal gains `; the keys it holds are …` / `; it holds no key at all`.**
   - Assertions: `internal/websterengine/verifyfixer_test.go:116`, `internal/weaverengine/render_test.go:149`, `internal/weaverengine/run_test.go:197` and `internal/gateslot/config_test.go:1021,1027` all match on the unchanged prefix `gates holds no key "<k>"`.
   - The gatecli refusal path is `run.go:444` (`gateKeyRefusal`).
   - `gateslot/doc.go`: updated. Covered.
4. **`DarnInputs.MeasureCall`, a new required field, and the `makeChangeList` signature.**
   - Every `DarnInputs{` literal is updated: `internal/loomcli/darnwiring.go:321`, `internal/shuttleparity/seats.go:201` and `internal/darnengine/spec_test.go:27`.
   - Refusal row added; `darnengine/doc.go` updated. Covered.
5. **New `measure_call` stencil marker.**
   - `contracts/stencils/markers_test.go` marker set, the `stencils.go` doc and the stencil header comment are all updated.
   - Preflight's `CheckMarkers` is filled through `darnTable`. Covered.
   - Stencil wording: the only rule text changed is "every tier the key lists" → "every tier the call runs", which follows the new call. No other rule was dropped or weakened.
6. **`Config.MeasureCall`, whose fallback spells `lym gate test --tier 0 [<its package or test project>] -- <args>` into the darn prompt.**
   - `pattern/PATTERN-test-speed.md:233` says "A site or stencil names a gate key, never a tier list…".
   - `pattern/PATTERN-gate-slot.md:116` calls `gateslot.FailedTestsCall` "the one call a site spells with tiers and targets".
   - Neither is updated. Not covered (finding 2).
7. **Env-differs line** (`envOverrideLine`, `testRequest.sessionEnv`).
   - Every request reaches `runTest`, which sets `sessionEnv` from `os.Environ()` (`run.go:121`): the CLI, `RunGate` and red-first. The detached runner inherits its starter's environment (`detachfront.go:171`).
   - Tests that assert exact gate output: the integration suite drops the session's `GOFLAGS` (`test_integration_test.go:403`). `runTier` unit tests leave `sessionEnv` nil, so they get no line.
   - `doc.go`: updated. Covered.
8. **Commit messages and departures.**
   - Each of the three feature commits carries a `Root-Cause:` that names the defect its diff removes: the console-only counter, the hard-coded `darn-write` key and target word, and the unreported env overlay.
   - The two `fix:` commits repair the branch's own work.
   - There are no `Pattern-Departure:` or `Loosened-Check:` trailers. The `GOFLAGS` drop in `startGate` changes the environment the test feeds the gate; it widens or removes no assertion.
9. **Declared PATTERN changes:** the directive declares none, so there is nothing to check.

**Findings**

1. **BLOCKING · design · `internal/gatecli/tierrun.go:487`**
   - **Summary:** the results arguments go to every `vstest` tier whose command holds `{args}`, whatever program it runs. That breaks the `dotnet run … -- {args}` timing tier `docs/dotnet-hub.md` recommends: BenchmarkDotNet is handed `--results-directory`, `--logger trx;…` and `--blame`. It also hands Microsoft.Testing.Platform commands spellings they do not take.
   - **Docs:** `docs/dotnet-hub.md` says nothing of the injection in either its Tiers or its timing-tier section.
   - **Evidence:**
     - `tierrun.go:486-493`: the condition is only `counter.(*vstestCounter)` and `strings.Contains(run.tier.Command, "{args}")`.
     - `docs/dotnet-hub.md`, Tiers: "A tier's command holds `{targets}`, `{filter}` and optionally `{args}`".
     - `docs/dotnet-hub.md`, timing tier: `command: dotnet run -c Release --project NORCE.Models.Benchmarks -- {args}` with `counter: vstest`.
     - `_lym/landing/summary.md`, "Open risks", names only `dotnet vstest` and MTP.
   - **Fix:** gate the injection on a command that runs `dotnet test` in VSTest mode, or behind a tier setting. Document the behaviour, and the tiers it skips, in `docs/dotnet-hub.md`.
2. **MEDIUM · consistency · `internal/gateslot/tiers.go:436`**
   - **Summary:** the `MeasureCall` fallback renders `--tier 0` and a target word into the darn prompt. The PATTERN backgrounds still say a site or stencil names only a gate key, and that `FailedTestsCall` is the one call a site spells with tiers and targets. Neither background was updated, and no `Pattern-Departure:` trailer explains the gap.
   - **Evidence:**
     - `internal/gateslot/tiers.go:442-447`.
     - `pattern/PATTERN-test-speed.md:233`.
     - `pattern/PATTERN-gate-slot.md:116`.
     - The commit `f305676b` message carries no trailer.
   - **Fix:** amend both pattern background lines to admit the measuring call's `--tier 0` fallback, as the board entry licenses, or have the parent record a decision.

**Tool calls**

1. A grep for the refusal prefix, the `DarnInputs` literals, old darn-call wording, docs that mention `vstest`, and the PATTERN tier-call lines. My context did not have their other readers.
2. A grep of `docs/dotnet-hub.md`, `docs/overview.md` and `PATTERN.md` for vstest, darn and env wording, to find out whether the docs describe the changed behaviour.
3. Read the `docs/dotnet-hub.md` Tiers, gate-keys and timing-tier sections. These are the operator docs for `{args}` on vstest tiers, which my context lacked.
4. Listed `internal/gatecli` for the detached front and runner files, to trace where a capped call's `os.Environ()` comes from.
5. A grep of `detachfront.go` for how the runner's environment is built and how calls join a run. The runner is the process that reads `sessionEnv`.
6. A grep for production readers of the run record's `Command`, to see whether the temp path that differs on every run breaks a carry or a join key.
