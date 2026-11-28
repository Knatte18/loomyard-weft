```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. I ran each function and branch the diff adds or changes on a concrete case and checked the result.

**Case list (function or branch, case run, verdict):**
- `LoadConfig` on `attempt_timeout_min: 0`: refused, names the key and is marked invalid. When the key is absent, the template default of 90 fills it. Correct.
- `RunDeps.attemptTimeout` with `AttemptTimeoutMin` 90: returns 90m. With 0, from a RunDeps not built through LoadConfig, it returns 0 and shuttle falls back to `run_timeout_min` (`spec.go:261`, `attach.go:385`). Correct. Every production RunDeps takes its Config from LoadConfig (`entries_weaver.go:58`, `battencli/wire.go:824`).
- `attemptSpec` with Timeout set: lands on `Spec.Timeout`. Correct.
- `runBatch` spawn: the deadline is the spawn instant plus Timeout (`run.go:820`). Correct.
- `resumeAttempt` probe and attach: the attach deadline is the attach instant plus Timeout, not the creation time (`attach.go:342`). That matches the `attemptLaunch.Timeout` comment. Correct.
- `timedVerify`:
  - On a pass, one gate span from start to end. Correct.
  - When verify returns an error, a span is still appended and the error is passed through. Correct.
  - A pause-hold wait inside Verify falls inside the span. Webster's verify seam also runs the hold inside its closure's gate span (`verifygate.go:269`, `wait.go:1905`). Consistent.
- `runVerify` with `GateConfig` nil: errors before any verify call and appends no span. Correct.
- `finishRun`:
  - On the done return, the deferred assignment to the named result sets the spans. Correct.
  - The `stuck(...)` returns and `verifyStop` also go through the deferred assignment, so they carry the spans. Correct.
  - Unjudged first run: one gate span, then a stuck result. Correct.
  - A fixer whose `fixerSpec` or `SpawnOrAttach` fails: a rework span is still appended, then a stuck result (see the finding).
- Re-entry `re.Verify` routing to `finishRun` (`run.go:196`): spans are recorded. A fixer that `resumeFixers` settled earlier gets no span, as the change description states. Correct as documented.
- `stoppedBatchGatesPass`, which runs batch gates at re-entry without timing: these are not the plan-level verify, so no span is expected. Correct.
- `writeSummary`:
  - Digest nil: both lists read `none`.
  - Digest with two deviations: backticked and joined with ", ".
  - Two departures: joined with "; ".
  - Each route to done sets a digest: the normal path through `record.go:197`, resume through `resume.go:513` and re-entry through `recordDoneWithoutSpawn`.
  - Correct.
- `weaverVerifySpans`: with nil spans, `gateSpansOf` loops over an entry that has no spans and returns nil. With spans, each maps to `Gate: "verify"` with its kind. Correct.
- `WeaverProducer.Call`: the done and stuck results carry the spans. On the stuck path, `claimConflict` receives the pointer with the spans already set and only sets Reason and Reroute on it (per its doc). Correct.

**Findings:**

1. **LOW, design.** Location: `internal/weaverengine/run.go:772-774`, in `finishRun`.
   A verify fixer that never spawned still records a rework span.
   `fixStart` is taken before `runFixer`, and the span is appended unconditionally. So when `fixerSpec` fails (`run.go:817-819`) or `SpawnOrAttach` fails (`run.go:822-824`), the row gets a near-zero rework span for a fixer that never ran.
   That contradicts the `RunResult.VerifySpans` doc ("one rework span per verify fixer, from its spawn to its end", in `run.go` in the diff's RunResult hunk).
   Evidence: `run.go:816-830` and the diff hunk at `run.go:769-779`.
   Fix: append the span only when the spawn succeeded, for example by timing inside `runFixer` after `SpawnOrAttach` returns.

Tool calls:
1. Read `run.go:180-225`, `runFixer` and `awaitFixer`, and grepped the wiring Config. The manifest lacked `finishRun`'s callers in context and `runFixer`'s body.
2. Grepped the weaverengine LoadConfig and RunDeps construction sites, to confirm that production Config always passes LoadConfig; the manifest lacked these.
