```yaml
lens: code-affected-set
outcome: reported
```

Task restated: the code-affected-set lens, covering what the diff forces outside the lines it touches.

**Set per change**

1. New `OutcomeAPIError` / recorded `"api-error"` outcome.
   - Covered: `attach.go` `isTerminalOutcome`, `waitshadow.go`, the `completionsignal_enforcement_test.go` markers, and the `call.go`/`tackler.go` switches.
   - Not covered: `carryover.go:118-126` `unfinished()`. It reads a record's outcome and carries over instructions for a successor run of the same role when the predecessor ended stopped, died or timed out. `api-error` falls through to false. The arbiter and tackler (the only specs that can record it) are respawned per attempt under the same role key, so the next attempt after an api-error does not inherit the operator's instructions.
   - Not covered: the `internal/shuttleengine` doc comments on `unfinished` say "stopped, died or timed out".
2. New hand-on/hand-back cause `api-error`.
   - Not covered: `contracts/orchrole/orch-role.md:135` lists the hand-on causes as `timeout`, `died`, `missing`, `unparseable`, `precondition`. It is the orch role's contract for classifying a review escalation, and the diff does not add `api-error`. The orch can read the new hand-on as outside the listed cases.
   - Covered: `sheddriver/doc.go`, `shedarbiter/doc.go`, `drivehandback.go` (the cause is printed generically in the report), `seam.go` (Cause passes through).
3. `Spec.APIErrorRetries` and the "settlers only opt in" behaviour.
   - Covered: `spec.go` doc, `shuttleengine/doc.go`, both settler specs and their `TestSpec`/`TestTacklerSpec` rows.
   - Not covered: `docs/overview.md`'s shuttle entry says "`Stop`-hook completion … classified into three outcomes — `done`/`died`/`timeout`" and describes a held turn end as always polling to the deadline. A fourth outcome now exists, and the diff touches no file under `docs/`.
4. The Completion Signal rule.
   - Covered: the AST tripwire scan adds `apierrorretry.go`, with `OutcomeAPIError` as a marker.
   - Not covered: `pattern/PATTERN-shuttle-provider-seam.md:54` enumerates "the negative answers" as `OutcomeDied`, `OutcomeTimeout`, `runOutcomeStopped`, a mechanism-failure error, and `verdictRespawnEligible`. Line 56 lists the helpers through which `allOutputFilesExist` is reached, and `retryAPIError` is not among them. The diff changes the enforcement test but not the entry's text. `PATTERN.md` itself needs no change.
5. `HandBackQuestionCause` and the changed hand-back question text.
   - Covered: both `handOn` and `handBack` use it, and the arbiter row asserts the question.
   - Gap: the tackler row (`tackler_test.go:283-288`) has no question assertion. The shared helper is covered through the arbiter row, so I report this as LOW.
6. `Pattern-Departure:` trailers and declared PATTERN changes.
   - No declarations in the directive and no trailers in the three commit messages, so there is nothing to land and nothing to check.
   - The commit messages are correct as written.

**Findings**

- **F-A1 | MEDIUM | consistency** | `contracts/orchrole/orch-role.md:135`: the hand-on cause list omits the new `api-error` cause. The diff adds a hand-on with that cause that reaches the orch, and the stencil defines which hand-on causes count as faults. Fix: add `api-error` to the list. Evidence: `orch-role.md:135`, `shedarbiter/call.go:268-269`.
- **F-A2 | MEDIUM | design** | `internal/shuttleengine/carryover.go:118-126`: `unfinished` does not treat the recorded `api-error` outcome as an unfinished run. The next settler attempt of the role drops the previous attempt's carried and operator instructions, unlike after a timeout or death. Fix: add `string(OutcomeAPIError)` to the case at line 120 and update the doc comment at line 117. Evidence: `carryover.go:117-126`, `attach.go:608-615` (api-error is recorded as terminal).
- **F-A3 | LOW | consistency** | `pattern/PATTERN-shuttle-provider-seam.md:52-59`: the Completion Signal entry's list of negative answers and of helper sites does not name `OutcomeAPIError` or `retryAPIError`. The enforcement test now audits both, so the text and the test disagree. Fix: add `OutcomeAPIError` to the negative-answer list and `retryAPIError` to the helpers sentence.
- **F-A4 | LOW | consistency** | `docs/overview.md`, the shuttle bullet: "three outcomes — `done`/`died`/`timeout`" is now four, and it says a held turn end always waits to the deadline. Fix: name `api-error` and the `APIErrorRetries` opt-in there.
- **F-A5 | LOW | scope** | `internal/sheddriver/tackler_test.go:283-288`: the tackler api-error row asserts cause, verdict and stop count only, not that the question or reason names the error as the summary claims. Fix: add a `wantQuestion` assertion for the tackler row, as the arbiter test has.

**Tool calls**
1. A grep of non-test Go and doc files for other readers of run outcomes (`OutcomeTimeout`, `runOutcomeStopped`, `"timeout"`) and docs listing the outcomes. My context did not hold readers of the recorded outcome string outside `wait.go`, `attach.go`, `stop.go` and `engine.go`.
2. A Read of `carryover.go:95-145`, to confirm what `unfinished` does with a recorded outcome. The first grep showed it matching on `OutcomeDied` and `OutcomeTimeout` but did not show its body.
