```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change in the diff, find what reads or asserts it and report what the diff leaves wrong.

**Per change, the set found:**

- **New `weaver.attempt_timeout_min` key** (`Config.AttemptTimeoutMin`, `attemptTimeoutMinKey`, `ConfigSpec` categories, template.yaml, `LoadConfig` refusal, doc.go).
  - Readers: `LoadConfig` via loomcli wiring, and `RunDeps.attemptTimeout` at `runBatch` and `resumeAttempt`. Covered.
  - Tests: `config_test.go` (template default and zero-value row) and `run_fixture_test.go` (`AttemptTimeoutMin: 45`). Covered.
  - Docs outside the package: no doc lists Weaver's config keys. `contracts/specs/implementer-spec.md` and `docs/overview.md` name none of them (grep for `fixer_timeout_min` and `verify_gate`). Nothing left stale.
  - Rule documents: PATTERN-config-key-names is not met in doc.go (finding 1).
  - PATTERN-config-layers: a new key whose default sits in the template needs no move table or reinterpretation. Covered.
- **`attemptLaunch.Timeout` / `attemptSpec`.** Both callers, `runBatch` and `resumeAttempt`, set it. `shuttleengine` defaults a zero value to `run_timeout_min` at spawn and at attach (spec.go:261, attach.go:385). An attach's deadline is attach time plus Timeout (attach.go:342). Covered.
- **`RunResult.VerifySpans`.**
  - Readers: `shedadapters/weaver.go` only. `loomshed/weaver_test.go:186` builds a `RunResult` without the field, which still compiles. Covered.
  - Downstream: `shedengine` passes `OutputPointer.GateSpans` into the history (run.go:366, reroute.go:49,59,70). `loomcli/checklist.go:735ff` renders spans by gate name and counts rework spans as "refused n rework", which reads sensibly for one rework span per fixer. Covered.
- **`VerifyGateName = "verify"`.** It matches Webster's `verifyGateName` (websterengine/verifygate.go:29), so the checklist groups both rows under `verify`. The name is a named constant, as PATTERN-code-conventions asks. Covered.
- **`runVerify` signature and `timedVerify`.** `finishRun` is the only caller. `stoppedBatchGatesPass` (resume.go) calls `deps.verify()` directly, without `timedVerify`; the decision record scopes the spans to the plan-level verify only. Covered.
- **`finishRun` named results and the deferred span assignment.** Callers are run.go:197 and run.go:217; the deferred assignment sets the spans on every return path. Covered.
- **summary.md batch sub-lines.**
  - Readers: `summaryparser.ParseData` validates only the title heading (summary.go:53-83), so the added lines break no parser. `CommitMessage` and `LandingMessage` carry the body verbatim.
  - Existing assertions: run_test.go:136-138 (prefix and section absence), 721 and 739 (substring checks). All still hold.
  - Wording against Webster: the departures join on "; " as `websterengine/render.go:577` does. The deviation and departure lists match `webster-template-master.md:118-127,352`. Covered.
- **Digest fields as the summary's source.** Both writers of the digest are on the route to done: `record.go:191-198` and `recordDoneWithoutSpawn` (resume.go:509-513). Covered.
- **Test fixtures** (`fakeGit.commit` departures, `fakeIndex.outsideTargets`, the `paneDies` script). Every existing call of `commit(subject)` still compiles with the variadic parameter. Covered.
- **Commit messages and departures.** No commit in the range has a `Pattern-Departure:`, `Loosened-Check:` or `Root-Cause:` trailer.
  - The old `MachineryLossRetriesOnceOnTheSameRungThenClimbs` assertions move into the `losses` table: `Outcome` is lost and `Number` is i+1, as tight as before.
  - The `DonePointsAtTheSummary` and stuck-reason assertions are kept and widened.
  - Nothing is loosened.
- **Declared PATTERN changes.** The directive declares none. Nothing to check.

**Findings:**

1. **BLOCKING, consistency.** `internal/weaverengine/doc.go:93` (the added `attempt_timeout_min` in the key list). The diff adds the new key to the package doc's list by its bare name, `` `attempt_timeout_min` ``. PATTERN-config-key-names (PATTERN.md:17) requires a config key named in a comment or doc to be written in full, `&lt;module&gt;.&lt;dotted path&gt;`. The diff's own next line, doc.go:96, writes `weaver.attempt_timeout_min`, so the doc now names the same key both ways. The `LoadConfig` doc in the same diff (config.go:33-34) was rewritten to use full names. Evidence: diff 002 lines 93-96, PATTERN.md:17.
   - Fix: write the added name in the list as `weaver.attempt_timeout_min`.

Tool calls:
1. Grepped for builders of `weaverengine.Config`, users of `RunResult`, gate_spans docs, the summaryparser and docs naming weaver keys: the manifest held no consumers outside the package.
2. Read `summaryparser.ParseData` and the summary assertions in `weaverengine`'s tests: needed to judge whether the new summary lines break a reader or an existing assertion.
3. Read `run_test.go:128-139` and grepped for other `GateSpans` readers: needed the existing summary assertion and the consumers downstream of the spans.
4. Read the `GateSpans` use in `loomcli/checklist.go` and `shedengine/reroute.go`: needed to see how the Weaver verify spans are rendered and carried.
