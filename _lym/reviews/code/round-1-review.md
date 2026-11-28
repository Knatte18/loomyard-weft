---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    origin: lens:code-affected-set
    location: internal/weaverengine/doc.go:4
    summary: The package doc's weaver.yaml key list adds the new key by its bare name, against PATTERN-config-key-names, and line 6 names the same key in full.
    fix: "In internal/weaverengine/doc.go line 4, the line the diff changed, write every key in full as PATTERN-config-key-names asks: `weaver.ladder`, `weaver.attempt_timeout_min`, `weaver.fixer_timeout_min` and `weaver.verify_gate`. Lines 5, 7, 8 and 9 still name `ladder`, `fixer_timeout_min` and `verify_gate` bare. Write those in full too, so the paragraph names each key one way."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/weaverengine/run_test.go#spanKinds
    summary: The verify span tests assert only span kinds, never their Start and End. The fixture clock never advances, so the measured intervals of board point 3 are uncovered.
    fix: "In internal/weaverengine/run_fixture_test.go, make the verify fake and the fixer fake advance the fixture's fakeClock by a fixed step each, such as one minute per verify run and five minutes per fixer, before they return. Check first that fakeClock exposes a way to advance it, beyond Sleep. In the fixer-clears test and the exhausted-ladder test in internal/weaverengine/run_test.go, assert each span's Kind, Start and End against literal instants derived from the fixture's start time and those steps. Replace or extend the spanKinds comparison there."
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/weaverengine/run.go:772
    summary: finishRun appends a rework span for a verify fixer whose fixerSpec or SpawnOrAttach failed, so a fixer that never ran shows a near-zero span. The VerifySpans doc says each rework span runs from a fixer's spawn to its end.
    fix: "Record the rework span only for a fixer that spawned. One way is to move the timing into runFixer in internal/weaverengine/run.go: take the start after deps.Fixer.SpawnOrAttach returns without error, and append the span after awaitFixer returns, through a spans pointer like the one runVerify takes. Then remove the unconditional append after runFixer in finishRun."
  - id: F4
    severity: LOW
    class: consistency
    origin: handler
    location: internal/weaverengine/config.go:24
    summary: The AttemptTimeoutMin doc and template.yaml's comment say the bound counts from spawn. An attempt that a crash left running and the next call attaches to gets a fresh full bound from the attach (shuttleengine attach.go:342).
    fix: "In internal/weaverengine/config.go, reword the AttemptTimeoutMin comment to say the bound counts from the strand's spawn, or from an attach for a strand a crashed call left. The attemptLaunch.Timeout comment in strand.go already says this. Make the same change to the attempt_timeout_min comment in internal/weaverengine/template.yaml:5, keeping that comment on one line."
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/weaverengine/resume_test.go:240
    summary: The change description says a verify fixer that the next call attaches to while it settles a crash adds no span. No resume row asserts result.VerifySpans after such an attach.
    fix: "In the TestRun_Resume row of internal/weaverengine/resume_test.go that attaches to a live verify fixer and then runs the verify, assert that result.VerifySpans holds only the gate spans of the verify runs that follow, with no rework span for the attached fixer. Compare with spanKinds against a literal list."
---

### [BLOCKING] The doc names a new config key bare, against PATTERN-config-key-names

**Location:** `internal/weaverengine/doc.go:4`
**Issue:** The diff changes the key list to add `attempt_timeout_min` without the `weaver.` prefix.
PATTERN-config-key-names requires a key named in a comment or doc in full, `<module>.<dotted path>`.
Line 6 of the same paragraph names the key in full as `weaver.attempt_timeout_min`, so the paragraph now names it two ways.
The diff also rewrote the `LoadConfig` doc in `config.go` to use full names.

### [MEDIUM] Verify span tests never check the intervals

**Location:** `internal/weaverengine/run_test.go#spanKinds`
**Issue:** Each span assertion goes through `spanKinds`, which compares kinds only.
The fixture's clock moves only on `Sleep`, so every span starts and ends at one instant.
A `timedVerify` or `finishRun` that swapped Start and End, or started the fixer span after the fixer returned, passes every test.
Board point 3 asks the row to record the time the verify took, so the intervals are the behaviour under test.

### [LOW] A fixer that never spawned records a rework span

**Location:** `internal/weaverengine/run.go:772`
**Issue:** `fixStart` is read before `runFixer`, and the span is appended whatever `runFixer` returns.
When `fixerSpec` or `deps.Fixer.SpawnOrAttach` fails (`run.go:818-824`), the stuck result carries a rework span for a fixer that never ran.

### [LOW] The attempt timeout's comments say "after spawn", which an attach does not keep

**Location:** `internal/weaverengine/config.go:24`
**Issue:** `shuttleengine`'s attach computes its deadline as the attach instant plus `Spec.Timeout` (`attach.go:342`).
So a strand that a crash left running gets a new full bound at the attach in `resumeAttempt`.
The `attemptLaunch.Timeout` comment states this.
The `Config` field doc and the `template.yaml` comment count the bound only from spawn, so they disagree with it.

### [LOW] No test pins that an attached verify fixer adds no span

**Location:** `internal/weaverengine/resume_test.go:240`
**Issue:** The change description's design decisions say only the verify runs and fixers that `finishRun` starts are timed.
A fixer that `resumeFixers` attaches to is not one of them.
No row checks `result.VerifySpans` on that path, so a later change that timed the attach would pass.

## Rejected

- Webster's verify entry can carry `held` spans and Weaver's never does (raised as a question by code-scope). `GateSpanHeld` measures a gate entry pending and holding a shuttle run (`shuttleengine/gate.go:156`). Weaver's verify runs outside any shuttle gate, so it has no such interval. Pause-hold waits inside `verifytree.Verify` fall inside the gate span on both rows (`websterengine/verifygate.go:269`, `shuttleengine/wait.go:1905`).
- Weaver records the first run and the rerun of a failed verify as two gate spans, while Webster's closure records both in one. The intervals cover the same time and the totals agree, and the change description states the split. No defect.
- The `; ` join of several departures is unpinned (a NIT the code-tests lens did not raise). It does not reach a reportable severity.
