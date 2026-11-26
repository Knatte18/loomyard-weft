```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change in the diff I traced what reads, calls, asserts or documents it, and checked it against the repo rules.

**Per change, the set found:**

- **`gatefixer.Await` order (strand read before timeout) and new `DeadReasonInterrupted`**
  - `rowgate.go` `wait`/`note`: covered.
  - `gatefixer/doc.go`: covered.
  - `websterengine/verifyfixer.go:126` `awaitFixerStrand` (used by FixVerify and FixPlan): reaches the new code, but builds `Deps` with no `ShuttleCfg`/`AnchorRoot`. **Not covered** (F1).
  - `websterengine/verifyfixer.go:246-248` `FixVerifyAwait` doc still states the old order and reasons. **Stale** (F2).
  - `runplan/digest.go:22` "The three legal Digest.DeadReason values": a separate batch digest. No interrupted value reaches it, so it is fine.
  - `websterengine/classify.go:30-32`: a separate batch classifier, not affected.
  - `webster recoverbatch.go:923` special-cases only `asking`, so `interrupted` falls to generic dead. Fine.
- **`rowStore.count` → `budget`:** the only callers are in `rowgate.go` and the tests, all updated. Covered.
- **`waitThenRun` `replace` parameter:** all three call sites in `rowgate.go` updated. Covered.
- **`fixerOutcome.note` interrupted text:** read by `waitThenRun` and asserted in the `TestWrapRowGate` cap row. The text says "no replacement ran" even on the cap path, where one ran (F3).
- **`Bouncer.judgeCall` died path returns an error:**
  - `shedtransient.Class` classes `ErrNotStarted` as transient and `ErrAgentDied` as not, unless EAGAIN. Consistent.
  - The engine's producer-error path (`shedengine/run.go:401-406`): consistent.
  - The `shedadapters/doc.go` Bouncer convention text does not describe died judges. No stale claim.
  - The degradations tests were updated, and the earlier `OutcomeDied` rows moved to the new test. Covered.
- **`boxCapture` plain fallback:** callers are `send.go:263` and `:946`, `session.go:356`, `run.go:1387` and `interruptguard.go:77`. All take the new behaviour with no signature change. The doc on `boxCapture` was updated. No docs or specs name it. Covered.
- **`holdsOnlyAPIErrorPrompt` in `InputBoxText`/`IdleSession`:** reached by tender's conditioned send (`tendercli/deps.go:494`) through `boxCapture` and `InputBoxText`. Docs on both methods were updated. Covered.
- **`planPaneOOMNotices` one notice per cgroup:** its doc was updated and the `TestPlanPaneOOMNotices` fixture gained the shared-scope row. Covered.
- **Rule documents:**
  - No declared PATTERN changes, so nothing to land.
  - PATTERN-tolerant-records: `records.json` still goes through `internal/state`. Fine.
  - PATTERN-test-only-code: the new helpers are in `_test.go` files. Fine.
  - PATTERN-shuttle-provider-seam: the Claude-specific string stays under `claudeengine`. Fine.
  - PATTERN-cwd-resolution: affected (F1).
- **Commit trailers:** every fix commit carries `Root-Cause:`. No `Pattern-Departure:` or `Loosened-Check:` trailers. The removed `OutcomeDied` degrade rows are not a loosened check, because the new test pins that behaviour more strictly.

**Findings:**

- **F1. MEDIUM, design.** `internal/gatefixer/await.go:66-72` (`goneStrandReason`), reached from `internal/websterengine/verifyfixer.go:127`.
  - Webster's `awaitFixerStrand` builds `gatefixer.Deps` with no `ShuttleCfg` or `AnchorRoot`. For a gone strand, `goneStrandReason` then calls `FindRun` with a zero config and an empty anchor.
  - `runDirRoot` (`shuttleengine/rundir.go:71-73`) turns that into the relative path `.lym/shuttle`, resolved against the process cwd.
  - The result: the reason webster's fix-plan and fix-verify fixers get depends on the cwd. This conflicts with PATTERN-cwd-resolution ("engines get absolute paths").
  - The diff changes the shared `Await` without wiring the new dependency into its other caller. The webster rows of `TestAwait` / `TestFixVerifyAwait` do not pin either outcome.
- **F2. LOW, consistency.** `internal/websterengine/verifyfixer.go:246-248`. `FixVerifyAwait`'s doc still says the fixer is dead "checked in this order … asking; … timeout; or … strand is gone (DeadReasonDied)". It delegates to `gatefixer.Await`, which now reads the strand before the timeout and can also return `DeadReasonInterrupted` (`await.go:72-81`).
- **F3. LOW, consistency.** `internal/gatefixer/rowgate.go:49`, text read at `rowgate.go:257`.
  - The interrupted note says "and no replacement ran". `waitReplacingInterrupted` returns that same interrupted outcome after the bound is passed (`rowgate.go:280-281`), which is after one or more replacements did run.
  - `TestWrapRowGate`'s `AnInterruptionPastTheCapStandsWithItsNote` asserts two spawns and this note together (`rowgate_test.go:455-459`). So the message the writer gets contradicts the case the test pins.

Tool calls:
1. Webster's `FixVerifyAwait` doc, its dead-reason handling and the `runplan` dead-reason constants, to see if the Await change reaches webster.
2. A search of docs, specs and `shedadapters/doc.go` for the changed behaviours (dead reasons, cold-fixer note, bounce budget), to find stale docs.
3. Webster's `awaitFixerStrand` body, the remaining `DeadReason` readers and `docs/overview.md`, to confirm how webster builds `Deps`.
4. The `shedadapters/doc.go` Bouncer convention section, to check its claim about degraded judges.
5. `runDirRoot` in `shuttleengine/rundir.go`, to see where `FindRun` looks with a zero config and an empty anchor.
