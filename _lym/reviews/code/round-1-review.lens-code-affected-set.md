```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set. For each change, the set it touches outside its own lines.

**Per change:**

1. **Flaky section in the landing texts (`landingTexts`, `flakySection`).**
   - Set found: both `landUnderLock` call sites, the `Call` doc, `landingshed/doc.go` and `contracts/specs/loom-final-summary-spec.md`.
   - Set found: `summaryparser.Summary.LandingMessage` is the only other `LandingMessage` caller outside the tests. Finalize now builds its message and body through `landingTexts`.
   - Verdict: covered. The code, `Call` doc, package doc and spec all agree. `TestFinalize_FlakyGateFailureLands` pins the message.
   - Verdict: no stale `landingMessage` parameter is left, but see F1.

2. **`gateslot.TierRunFailures`, a new parser of the failure report.**
   - Set found: `internal/loomshed/verifygate.go:248` and `internal/landingshed/verifygate.go:304` and `:309`, plus the `gateslot/doc.go` entry.
   - Verdict: covered. `FailedTests` is not a listed sole parser in `pattern/PATTERN-sole-parsers.md`, so no PATTERN violation. The `gateslot` doc entry was added.

3. **Failure-report line `pkg + " " + test` now trimmed, so a `vstest` test appears as `  <name>:`.**
   - Set found: `gateslot.reportTestLine` and `FailedTests`, which parse that line, and `loomshed` at `verifygate.go:248`.
   - Verdict: covered by the `FailedTests` rows, which I did not re-run. A one-field line must not parse as a Go test. `TestTierRunFailures` covers the `TierRunFailures` side.
   - Verdict: `gatecli/doc.go`, `failurereport.go` and the `RunRecord.Failures` comment are updated.

4. **The `vstest` counter now returns failures, so the run record's `failures` holds `vstest` tests.**
   - Set found: the `RunRecord.Failures` comment and `gatecli/doc.go`.
   - Verdict: covered. The `FailedTest` consumers reached by grep are the two above, so nothing else reads the record's `failures`.

5. **The rerun in `verifyGate.check`, a new behaviour.**
   - Set found: `landingshed/doc.go`, the `check` doc and `pattern/PATTERN-gate-slot.md`.
   - Verdict: the PATTERN is not updated, see F2.

**Findings:**

- **F2 — LOW, consistency.**
  - Location: `pattern/PATTERN-gate-slot.md:105` and `:108`.
  - The Finalize bullet says Finalize runs the `landing` call through `Verify` and states no rerun. The diff adds a one-time rerun on the same head before any fixer, using `FailedTestsCall`.
  - The `FailedTestsCall` bullet at `:105` names only the next-arrival re-run as the site that spells tiers and targets. The Finalize rerun is a second such site.
  - No `PATTERN.md` entry is declared changed, so this is not BLOCKING. The background is simply left silent on a new behaviour.
  - Fix: add one sentence to the Finalize bullet saying a Finalize gate failure of tier and guard runs reruns once on the same head, and that a rerun passing is recorded as flaky.

- **F1 — LOW, consistency.**
  - Location: `internal/landingshed/finalize.go`, the `landUnderLock` call to `landingTexts`, together with the `mergeOpts.Message` reassignment at about the second `landingTexts` call.
  - The texts are recomputed after the second gate pass, which is right. The `summary` handed in was built before the gate, though.
  - The `Call` doc says "After a gate failure that passed when run once more…" It does not say the section is also in the GitHub merge body, although the spec and `doc.go` do.
  - Fix: name "the GitHub merge's body" in the `Call` doc line.

**Not found:**
- No stencil or spec names the removed failure-report wording "no failing test or error line". The grep hit only `failurereport.go`, which is in the diff.
- No `Pattern-Departure:` trailer is needed. The three commits carry none, and none of their changes is forbidden by a listed entry.
- The diff declares no PATTERN changes.

**Tool calls:**
1. `grep` of `pattern/` and the Go sources for `FailedTests` and `FailureReportHeader` consumers. My context lacked the full caller set.
2. `grep -rln` for `LandingMessage`, `landUnderLock`, the old report wording and `Regression check`. This checked for other call sites and docs.
3. Read `pattern/PATTERN-gate-slot.md:98-112`, grepped `PATTERN-sole-parsers.md` and `PATTERN-landing.md`, and grepped `finalize_test.go`. This checked whether the PATTERN backgrounds describe the changed behaviour.
