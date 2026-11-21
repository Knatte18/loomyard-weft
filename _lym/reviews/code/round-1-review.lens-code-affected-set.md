```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. I traced everything outside the changed lines that calls, reads or asserts what the diff changes.

**Per change, the set found:**

- **`AppendIntegrationTriage` takes a second `commandLogs` argument.**
  - Callers: `VerifyGateNotes.Apply` (`internal/websterengine/verifygate.go:128`) and the `summary_test.go` rows. All are updated. Covered.
  - Spec `contracts/specs/webster-spec.md:71-72`: updated. Covered.
  - `contracts/specs/loom-final-summary-spec.md:92` describes the section as "the flaky-failure section the verify gate appends for identities that passed on rerun". It is not updated (finding 3).
- **The `## Integration suite triage` section of `summary.md` gains a sub-list of verify command failures, named by log.**
  - Reader: `contracts/stencils/landing/landing-template-describe.md:35` tells the describer to name "the flaky and pre-existing identities it lists". It does not know about the new sub-list (finding 1).
  - `internal/websterengine/runlevel_git_test.go:209` asserts the section is absent, so it is unaffected. Covered.
  - `internal/weaverengine/verify.go:331-356` and `internal/weaverengine/run_test.go:621` belong to weaver's own section writer, independent of this code. Not forced.
- **`VerifyGateNotes.Flaky()` becomes the unexported `flakyFailures()`.**
  - No production or doc caller is left. The tests are updated. Covered.
  - `docs/benchmarks/test-redundancy.md:3368` quotes the old keep reason, but it is a measurement report and exempt.
- **`addFlaky(failures, log)`: its one caller, `verifygate.go:491`, is updated. Covered.**
  - The `logger.Warn` on the next line, `verifygate.go:492`, still logs every first-run identity under the `flaky` key, `verify-command` included (finding 2).
- **The friction note `webster-verify-triage` changes shape.**
  - No code parser reads it (grep found only tests).
  - It is written through `friction.EnsureDir` and `NotePath` as at the base, so `PATTERN-run-records` holds.
  - Spec `webster-spec.md:89-90` and `doc.go:991-993` are updated. Covered.
- **Warnings: a new command-failure warning line.**
  - Consumers are the run warnings via `Apply`, and the tests. Covered.
- **`docs/overview.md:766`: "a pass on the rerun records the identities as flaky".**
  - It is a coarse summary, and it stays true for test and package identities. Not left wrong enough to file.
- **Repository rules.**
  - No PATTERN changes are declared, and none land.
  - The new test is tier0 with `t.Parallel` and spawns nothing (`PATTERN-test-speed`). Covered.
  - The `"webster-verify-triage"` literal is unchanged from the base.
- **Commit messages.**
  - `5ceacc30` carries a `Root-Cause:` trailer naming the dropped Tail and log and the opaque ID listed as flaky. The diff touches exactly that.
  - The revert and reapply cancel out. There is no `Pattern-Departure:` or `Loosened-Check:` trailer.
  - The base test's exact `notes.Flaky()` equality is replaced by substring checks on the note plus exact warnings. That is judged under the tests lens, not here. Covered.

**Findings:**

1. **LOW**, consistency.
   - Location: `contracts/stencils/landing/landing-template-describe.md:35`.
   - Summary: The landing describer is told to name "the flaky and pre-existing identities" the `## Integration suite triage` section lists. The section now also lists verify command failures by first-run log, and the stencil gives no wording for them. A PR description can therefore present a verify-step failure as flaky, or drop it.
   - Evidence: `landing-template-describe.md:35`; `internal/websterengine/summary.go:57`.
2. **LOW**, consistency.
   - Location: `internal/websterengine/verifygate.go:492`.
   - Summary: The rerun-pass `logger.Warn` still logs `failureIDs(first)` under the `flaky` key, so a `verify-command` opaque failure is logged as flaky. That contradicts the new rule in `doc.go:993` and `triage.go:41`: "never as a flaky test".
   - Evidence: `verifygate.go:491-492`.
3. **NIT**, consistency.
   - Location: `contracts/specs/loom-final-summary-spec.md:92`.
   - Summary: The pointer still describes webster's section as listing "identities that passed on rerun". It now also lists verify command failures by log.
   - Evidence: `loom-final-summary-spec.md:92`.

**Tool calls:**
1. Grep across `.go`, `.md`, `.yaml` and `.tmpl` files for section, note, warning and `Flaky()` readers outside the diff. The manifest only covered the readers inside `websterengine`.
2. Grep of docs, contracts and doc.go files for flaky-triage descriptions, plus `doc.go:980-1000`. I needed the doc and spec readers the manifest did not cover.
3. Read `docs/overview.md:762-770`, `landing-template-describe.md:30-38`, and the diffstat of `PATTERN.md`, `pattern/` and `docs/`. I needed the context of the readers grep found, and to confirm no PATTERN change landed.
