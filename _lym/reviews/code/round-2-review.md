---
verdict: APPROVED
prior:
  - id: F1
    status: resolved
    answer: "internal/weaverengine/doc.go lines 4-10 now name every key in full: `weaver.ladder`, `weaver.attempt_timeout_min`, `weaver.fixer_timeout_min` and `weaver.verify_gate`."
  - id: F2
    status: resolved
    answer: The verify fake advances the shared fixture clock by verifyRunTime and the fixer fake by fixerRunTime, and the fixer-clears and exhausted-ladder tests compare whole spans against spanAt literals.
  - id: F3
    status: resolved
    answer: runFixer takes the span start only after deps.Fixer.SpawnOrAttach returns without error and appends the rework span in a defer, and finishRun no longer appends one itself.
  - id: F4
    status: resolved
    answer: The AttemptTimeoutMin comment and the attempt_timeout_min comment in template.yaml both say the minutes count from the spawn or from an attach after a crash, matching the fresh deadline in shuttleengine attach.go.
  - id: F5
    status: resolved
    answer: The resume row that attaches to a live verify fixer now asserts spanKinds(result) equals a literal list holding one gate span and no rework span.
---

The review range is `git diff 1ae03e17f62aa83fbd81bfa9dbadabad3645366e..HEAD`, read through the two diff pieces, with `git log` over the same range.
`lym gate test ./internal/weaverengine ./internal/shedadapters` passed: 1229 tests, none failed.

Against the amended board entry, the diff makes all four changes:
a per-attempt timeout under `weaver.attempt_timeout_min`,
the summary's per-batch deviations and departures,
verify `gate_spans` on the Weaver row,
and tests of a lost strand, a resume, a failed plan-level verify and the third rung.
This round found no new BLOCKING defect on a changed hunk, and every prior finding is resolved.

## Focus verdicts

- **Config key names in doc.go, config.go and template.yaml.**
  Not a defect.
  `doc.go` names each key in full, and so do the `LoadConfig` doc and the `attemptTimeout` doc in `verify.go`.
  The new `Config` field comments and the template.yaml line comments name no key.
  `ConfigSpec`'s doc still opens a sentence with a bare `verify_gate`, but that line is outside every hunk of the range, so it is no subject of this review.
  The test failure messages in `run_test.go` and `resume_test.go` that say "bounded by attempt_timeout_min" follow the convention the repository's tests already use, such as `run_test.go`'s existing "want fixer_timeout_min".
  I read PATTERN-config-key-names, which asks for rendering through configengine's key-name helper from named constants, as covering production messages and docs.
  An external test package cannot reach the unexported `keyName` helper, so I raise no finding on the test messages.
- **Verify span intervals pinned against independent instants.**
  Not a defect.
  `scriptedVerify.run` sleeps `verifyRunTime` and `fakeFixer.Await` sleeps `fixerRunTime` on the one clock the run's deps use.
  The two tests compare whole `GateSpan` values against `spanAt` literals counted in minutes from the epoch.
  A swapped kind, a reordered span or a mistimed Start or End would each fail `slices.Equal`.
- **runFixer records a rework span only for a fixer that spawned, from spawn to end, and the VerifySpans doc matches.**
  Not a defect.
  An error from `fixerSpec` or `SpawnOrAttach` returns before the start is taken, so no span is added.
  Once the spawn succeeds, the deferred append covers every later return, an `awaitFixer` error included, as runFixer's doc says.
  The start is taken after `SpawnOrAttach` returns, so the span leaves out the spawn's own start-up.
  The `RunResult.VerifySpans` doc says "from its spawn to its end", which is close enough that I raise nothing in an incremental round.
  The `finishRun` doc says "fixers it started", and the doc.go line says "one rework span per fixer".
  Within `finishRun` the two agree, since a fixer attached during resume settles before `finishRun` starts.
- **resume_test pins an attached fixer adding no rework span while the following verify runs record gate spans.**
  Not a defect.
  The row asserts `spanKinds(result)` equals `[GateSpanGate]`, alongside one attach and no spawn.
- **AttemptTimeoutMin and template.yaml comments against the spawn and attach paths.**
  Not a defect.
  `runBatch` and `resumeAttempt` both pass `deps.attemptTimeout()` into `attemptLaunch.Timeout`, which `attemptSpec` copies into `Spec.Timeout`.
  On attach, shuttleengine's `reconstruct` sets the deadline to a fresh `now + Timeout` (attach.go:339-342).
  So "from the strand's spawn, or from an attach for a strand a crashed call left" holds for both paths, and the template.yaml comment says the same.

## Rejected

- Bare `attempt_timeout_min` in test failure messages and the config_test row name: these follow the repository's existing test convention and fall outside the pattern's production-rendering clause, as argued above.
- `ConfigSpec`'s bare `verify_gate` in its doc: the line is untouched by the range.
- The rework span starting after `SpawnOrAttach` returns rather than before it: a precision point below BLOCKING, and an incremental round raises only BLOCKING findings.
