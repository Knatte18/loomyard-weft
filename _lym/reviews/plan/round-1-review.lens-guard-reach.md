```yaml
lens: guard-reach
outcome: reported
```

Guard-reach lens: for each guard, refusal, branch or test row a card adds, whether a real entry point can reach it.

**Per guard, reachable or not**

- **Card 1, `LoadConfig` refusing an `agent` that `modelspec.Parse` rejects:** reachable. A hub `sampler.yaml` with `agent: "opus[[bad"` reaches it, and `TestLoadConfig` has that row.
- **Card 3, `SessionSpec` refusing an empty `StencilsDir`, `Slug`, `WorktreeRoot` or `DecisionRecordPath`:** not reachable from production. The only caller is card 9's spec closure, and it always fills these fields. It counts as a fail-loud wiring guard, which the rubric exempts.
- **Card 3, `SessionSpec` refusing an unresolvable `agent`:** reachable. Card 1 rejects only a value `Parse` refuses, so a spec that parses but that the registry cannot resolve gets through.
- **Card 4, `Call` refusing an unset seam:** a fail-loud wiring guard, exempt.
- **Card 4, transient start failure as an error, any other as Stuck with Halt:** reachable through `StartSession`. Test rows exist.
- **Card 4, `Spec()` error as Stuck with Halt:** reachable through a registry-unresolvable `agent` or a missing stencil. Card 9's `wire` load rejects only a value `Parse` refuses. A test row exists.
- **Card 4, done judgement on entry with a live reopened strand (stop first):** reachable. Reed resume relaunches the persisted strand (`internal/reedengine/lifecycle.go:190` `planResumeLaunches`), so `sessionLiveness` still finds it by GUID (`internal/loomcli/landingdeps.go:718`).
- **Card 4, flag at the current stamp with a dirty checkout:** reachable. Card 10's verb refuses a dirty checkout at write time, but the tree can change after the flag is written. This happens both in the wait and on entry after a halt.
- **Card 4, clean checkout with `CheckRecord` findings as Stuck with Halt:** reachable. `DirtyPaths` reads only the code checkout, so a damaged record under `_lym` gets past that check and only `CheckRecord` catches it.
- **Card 4, dead unstopped strand as Stuck with Halt:** reachable when the operator closes the window.
- **Card 4, covered wake and the first uncovered wake that resumes:** reachable through `PauseCovered`.
- **Card 4, `RegisterSession` failure and failed record write after a start:** reachable through their seams.
- **Card 5, `samplerSessionEntry` refusing config keys, nil seams and empty record paths:** the config-key refusal is reached by the `bogus_key` row in `minimalEntries`. The nil-seam and empty-path refusals are wiring guards, and test rows exist.
- **Card 6, `applyVerifyAttempts` refusing a named row with no verify entry:** no longer reachable once `verifyGatedRows` keeps only rows that carry a verify gate. The card keeps it on purpose as a fail-loud guard, so it is exempt.
- **Card 7, `unsupportedChildRecipeRefusal`, the wording of the `normalizeTask` pawl refusal, and the project-with-`sampler` refusal:** reachable through the named tests.
- **Card 8, `CodePerchGate` refusing an unknown recipe:** reachable in its test. In production, `landingDeps` passes an embedded recipe name.
- **Card 8, `codeBudgetZero` and `rerouteReviewed` reading the perch off with `code_review` false:** reachable. This is the D3 acceptance.
- **Card 8, the `code_review: "yes"` row where both seams return the gate's evaluation error:** in the plan's final state this input is rejected upstream (finding 1).
- **Card 9, `wire` refusing a malformed `sampler.yaml`:** reachable, and `TestWire_SamplerRecipe` has the `opus[[bad` row.
- **Card 9, `openRecordWatch` returning nil when the watch cannot open:** reachable when the record's directory sits under a regular file. A test row exists.
- **Card 10, refusal on a recipe with no `SamplerSession` row:** reachable on a darn run.
- **Card 10, refusal when the current row is not Sampler-Session:** reachable. A halt keeps `CurrentProducer` (`internal/shedengine/run.go:401`), so a run halted at the row passes this check, as D6 wants.
- **Card 10, refusal with no record or no recorded strand:** reachable in two ways:
  - in the window between Record-Write's Done and the row's first record write;
  - after a non-transient start failure halt or a `Spec()` halt on first entry, which writes no record.
- **Card 10, refusal on a dirty worktree:** reachable.
- **Card 10, seam read error envelopes:** reached through fakes.
- **Card 11, the sampler strand with a label other than `--by operator` or `--by arbiter`:** reachable. The strand name comes from `agentname.StrandNameEnv` (`internal/loomcli/decision.go:861`).
- **Card 11, the sampler strand with `--by arbiter`:** reachable, and the existing arbiter branch refuses it first. Its way forward names `--by operator` (`decision.go:84`).
- **Card 11, reader error:** reached through the fake.
- **Card 11, strand not admitted:** falls through to the parent branch, which is reachable.
- **Card 11, admission while halted:** reachable, because a halt keeps `CurrentProducer` (`run.go:401`).
- **Card 12, `childAtWaitingRow` and `agentActivityFor` reading a running status at a listed row:** reachable while a step runs at the row. An unreadable status reads as before, and a test row exists.
- **Recurring sites:** no other production site lists the operator-wait rows. `internal/reportengine/timingspans.go:94` maps PR-Review only to a timing phase, which is not a guard.

**Findings**

1. **LOW, consistency, card 8 (`08-rows-keyed-wiring.md`): the `TestRerouteReviewed_ReadsTheCodeBudgetFromLoomYaml` row with `code_review: "yes"` uses an input that card 9's `wire` rejects first in the plan's final state.**
   - In the row, both seams return the gate's evaluation error.
   - But card 9 has `wire` load `sampler.yaml` strictly through `samplerengine.LoadConfig` and refuse a malformed file "before any row builds". So in a real run a non-bool `code_review` never reaches Finalize's `rerouteReviewed` or `codeBudgetZero`. They can only meet it if the file is edited during the step, after `wire` ran.
   - The error branch is a propagated error, so it may count as exempt. The row still pins a path that the upstream check in card 9 forecloses.
   - Evidence: `_lym/plan/08-rows-keyed-wiring.md` (the `"yes"` row), and `_lym/plan/09-sampler-wiring.md` (`wire` loads `sampler.yaml` strictly through `samplerengine.LoadConfig` and refuses a malformed file before any row builds).

No other guard-reach findings.

**Tool calls**

1. grep of `decision.go` for the `strandName` source and of reed for resume functions: my context lacked where the caller's strand name and a reed resume come from.
2. `reedengine/lifecycle.go` `planResumeLaunches` and `loomcli` `sessionLiveness`: to see whether a session reopened by `lym reed resume` keeps the GUID that card 4's live check reads.
3. grep of `shedengine/run.go` for a halt's `CurrentProducer`, and of `prContinueCmd`: to confirm that a halt keeps the current row, which the checks in cards 10 and 11 rely on.
4. grep of production code for other lists of the operator-wait rows: to find a guarded site recurring outside card 12.
5. `reportengine/timingspans.go` around `NamePRReview`: to judge whether that hit is a guard.
6. `decision.go` `decisionWayArbiter` text: to check that the arbiter refusal a sampler strand reaches names `--by operator`.
