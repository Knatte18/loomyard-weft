---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md (Decisions, Command source; Constraints, Told-Geometry)
    summary: the record never says what form the verify command takes on Deps, and never says what a plan read or parse failure does, so a construction-time string or a parse-error-as-empty would silently skip the gate
  - id: F2
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md (Decisions, Pending-verify marker and No command means no gate; Acceptance criteria)
    summary: the marker lifecycle leaves three cases without a stated outcome (marker present with an empty command, a marker or output-file write failure, Finalize reaching a leftover marker on a no-op merge-in)
  - id: F3
    severity: LOW
    location: _lyx/discussion/decision-record.md (Decisions, Execution; Acceptance criteria, webster unchanged)
    summary: the record does not say what webster passes for the shared runner's new context and output writer, or that the spawn and teardown log lines stay, so the unchanged-behaviour criterion has no stated means
  - id: F4
    severity: LOW
    location: _lyx/discussion/decision-record.md (Goal; Execution)
    summary: the kill-on-cancel claim is stronger than sh -c plus a context delivers; the record should state the bounded-wait form treadleengine already uses and accept orphaned grandchildren as a risk
  - id: F5
    severity: LOW
    location: _lyx/discussion/decision-record.md (Goal)
    summary: the Goal's first line promises to stop every tree a parent merge-in broke, while Open risks accepts breaks outside the plan's verify scope; the Goal should promise only what the plan verify covers
  - id: F6
    severity: LOW
    location: _lyx/discussion/decision-record.md (Goal, Auto-mode assumptions, Open risks)
    summary: incident history, four auto-mode assumptions that restate Decisions and Scope Out, and the output-file-size risk carry no requirement the plan writer needs and belong in support-log.md or nowhere
  - id: F7
    severity: NIT
    location: _lyx/discussion/decision-record.md (Notes for the plan writer, second bullet)
    summary: the only note that carries a requirement, that a stuck resolver outcome returns before the gate, is absent from Decisions
---

Reviewed against the Discussion-Review rubric and the writer stencil `loom-template-discussion.md`.
Every Notes claim was checked against the code (`publish.go`, `finalize.go`, `integration.go`, `planparser/sections.go`, `loomcli/landingdeps.go`, `sharedbootstrap.go`), and the incident was checked against git history (`854eb73fe`, its parents `777d3df99` and `777d93af7`).
No finding asks for rejected alternatives in the decision record, for a Notes section, or for call-site enumeration.

### [MEDIUM] Command form on Deps and plan-read failure are unstated

**Location:** `decision-record.md`, Decisions "Command source"; Constraints "Told-Geometry Invariant" and "`landingDeps` drift guard".
**Issue:** The record says the command is resolved at call time, and says it is told through `Deps` and filled in `landingdeps.go`.
It never says what shape that field has.
`landingDeps` runs once at bootstrap (`sharedbootstrap.go:308`, `arm.go:317`), performs no I/O by its own header contract, and runs before the plan exists on a fresh run.
A plain string field, the natural reading of "told the verify command", would be empty on every fresh run, and "No command means no gate" would then skip the gate with only a warning, which defeats the task silently.
The acceptance criterion on call-time pickup would catch it in test, but the Decisions should not leave the plan writer to rediscover it.
Separately, a call-time read can fail (plan missing or malformed), and no outcome is stated; folding that into the empty-command skip would also skip the gate silently.
**Fix:** State that `Deps` carries a closure that `internal/loomcli` fills to parse the plan when the gate runs, and that a read or parse error is a returned producer error, never a skip; only a parsed plan with no `## verify:` section is the warn-and-skip case.

### [MEDIUM] Marker lifecycle has unstated combinations

**Location:** `decision-record.md`, Decisions "Pending-verify marker" and "No command means no gate"; Acceptance criteria.
**Issue:** Three cases have no stated outcome.
(a) Marker present and the command empty (the plan's `## verify:` removed between runs): the marker rule says the gate runs, the empty-command rule says it skips, and nothing says which wins or what happens to the marker.
(b) The marker write or the output-file create fails: neither Stuck nor a returned error is named.
(c) `Finalize` reaching a marker left by `Publish` on a no-op merge-in: the Decision covers it generically, and the case is reachable (a verify-Stuck `Publish` whose resume ends Done through the merged-PR branch of the approval check never re-runs the gate), yet only `Publish` has a marker-present acceptance criterion.
The spawn-failure criterion also does not say the marker remains, though the Decision implies it.
**Fix:** Add to Decisions: an empty command skips the gate whether or not a marker exists, running, writing and removing nothing, and the warning names a pending marker when there is one;
a marker or output-file write failure is a returned error, like a `CommitStatus` failure.
Add acceptance criteria for `Finalize` with a no-op merge-in and a marker present, for the empty-command-with-marker case, and "the marker remains" on the spawn-failure case.

### [LOW] The shared runner's contract for webster is unstated

**Location:** `decision-record.md`, Decisions "Execution"; Acceptance criteria "webster's bisect and integration behaviour is unchanged".
**Issue:** The shared form gains a context and an output writer.
Webster's only caller chain (`LocalizeIntegrationFailure` → `bisect` → `checkoutAndVerify` → `runVerifyCommand`) holds no context, and today's runner discards output.
The record does not say what webster passes, so a plan writer could thread a live context into the bisect (a behaviour change: cancellation mid-bisect) or start capturing output.
It also does not say the spawn and teardown log lines stay; `runverify_test.go` asserts them and the Live-Substrate Spawn Observability invariant requires them.
"Integration behaviour" is also loose: the runner serves only the bisect that localizes an integration failure, since the integration verify itself runs in the integration fork.
**Fix:** State that webster passes a background context and a discarding writer, that the shared runner keeps the spawn and teardown log lines, and name the bisect localization as what stays unchanged.

### [LOW] Kill-on-cancel claim overstates what the runner delivers

**Location:** `decision-record.md`, Decisions "Execution".
**Issue:** "Killed when the producer's context is cancelled" through `sh -c` kills the shell process.
A chained verify line (`a && b`) runs its commands as the shell's children, which a context kill does not reach, and a non-file output writer can leave `Wait` blocked on the copy goroutine.
`internal/treadleengine/gate.go` already settles the same problem with `exec.CommandContext` plus a bounded `WaitDelay`.
**Fix:** State the bounded-wait form as the requirement, and add orphaned grandchildren of a cancelled verify as an accepted Open risk.

### [LOW] The Goal's headline promise exceeds the design

**Location:** `decision-record.md`, Goal.
**Issue:** Focus entry 1, settled: the design does close the motivating incident.
The broken call sites (`internal/loomcli/wiring_commitstatus_test.go` and its `-tags integration` sibling, brought in from `777d93af7`) sit in `internal/loomcli`, which the task side itself changed (25 files), so the plan template's coverage rule (every targeted package, build-tagged tests included) required the plan's verify to run them.
The coverage requirement binds the plans being gated, not this task's implementation, so it stays out of Decisions and Constraints; the Command-source rationale already cites it, and the Open risk is a genuine accepted residual.
The Goal's first line, "Stop the landing producers from pushing or landing a tree that a parent merge-in broke", is broader than that, and contradicts the Open risk.
**Fix:** Scope the Goal to breaks the plan's `## verify:` covers.

### [LOW] Deliberation and restatement in decision-record.md

**Location:** `decision-record.md`, Goal, Auto-mode assumptions, Open risks.
**Issue:** Completeness-before-leanness, applied per item:
the incident's PR number, merge SHA and per-command results are history, and support-log Interview turn 1 already holds the PR and SHA; the one requirement they carry (a merge can compile and still break tests) is a single sentence.
Four of the five Auto-mode assumptions restate a Decision or a Scope Out line (command source, timeout, fix session, presence-only marker); the timeout bullet's `go test` reasoning is deliberation, already in support-log Interview turn 11.
Only "Finalize is in scope although the board names only Publish" records a real departure from the board.
The "Output file size" risk restates the Output-capture Decision (overwritten, never tracked).
The other three Open risks stay: each names an accepted residual the Decisions do not.
**Fix:** Cut the Goal's incident detail to one sentence and move the command-by-command detail to the support log; reduce Auto-mode assumptions to the Finalize departure plus one line pointing at the Question ledger; drop the output-file-size risk.

### [NIT] A requirement lives only in Notes

**Location:** `decision-record.md`, Notes for the plan writer, second bullet.
**Issue:** Focus entry 6, settled: every concrete Notes claim is accurate (the gate sits between `Publish.Call` step 4 and step 5; `mergeInStep` is called from `Finalize` steps 2 and 5; `runVerifyCommand` takes no context and runs with no stdout or stderr set; `planparser.Plan.Verify` is the `&&`-chained line; `planparser.PlanDir(location.AnchorPath())` is how loomcli reaches the plan in `wiring.go`).
One note carries a requirement: a stuck resolver outcome returns before the gate, so an aborted merge runs no verify and writes no marker.
Notes are optional and non-binding, so that belongs in the Trigger Decision.
**Fix:** Add it to the Trigger Decision; keep the note as orientation.
