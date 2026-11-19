---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/creelcli/slackbridge_test.go:19
    summary: Readiness tests omit EXTERNAL refusal and operational failure propagation.
    fix: Add rows to TestStore_PostToTheSlackRelay for an EXTERNAL sender refused without a journaled post, and to TestSlackBridge_RelayPostReady for EXTERNAL refusal and errors from geometry, beam discovery, forwarding reads and relay-lock inspection. Assert the propagated error and absence of a successful readiness result in the existing tables.
    origin: lens:code-tests
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/creelengine/verbs.go:193
    summary: The relay readiness check misses post --reply targeting SLACK.
    fix: Apply the relay readiness policy in Store.Reply when the original sender is the local Slack relay, before appending the reply or marking the original answered. Refuse unavailable delivery with its state and way forward, or propagate the requested delivery result through VerbReply and the CLI. Add a regression row for answering Slack mail after the button turns off or the relay stops.
    origin: lens:code-correctness
  - id: F3
    severity: MEDIUM
    class: scope
    location: internal/creelengine/verbs.go:212
    summary: A down relay returns neither success contract permitted by the board entry.
    fix: For a down relay, implement either the permitted refusal naming lym slack start before storage, or an OK response carrying delivered false and its reason through the daemon and CLI. Update the existing down-relay assertions and corresponding documentation to match the selected contract.
    origin: lens:code-scope
  - id: F4
    severity: LOW
    class: consistency
    location: _lym/landing/summary.md:20
    summary: The summary incorrectly describes the Slack turn hook as an unchanged program sender.
    fix: Correct the program-exemption paragraph in the landing summary to say that program reminders remain exempt while the Slack turn hook signs as an orch strand and therefore traverses the new readiness check.
    origin: lens:code-affected-set
---

### [BLOCKING] Readiness coverage omits external callers and failures

**Location:** internal/creelcli/slackbridge_test.go:19

**Issue:** The new bridge table supplies only successful geometry and readable state.
Its non-orch case is a strand, and the store table contains only strand and program senders.
Changing the store to bypass EXTERNAL would leave both tables passing.
The geometry-error row in the existing bridge test calls `SendReady`, not `RelayPostReady`.
The new operational error returns therefore also lack readiness coverage.
`PATTERN-test-speed` requires coverage of added behavior and applies the testing skills' boundary-error rules.
The review instructions make violations of listed constraints BLOCKING.
This finding concerns missing behavioral assertions, not a build or test failure owned by the mechanical gate.

### [MEDIUM] Replies to Slack still succeed silently

**Location:** internal/creelengine/verbs.go:193

**Issue:** Slack operator mail arrives from the relay program through `internal/slackengine/inbound.go:765`.
After the button turns off, an orch can answer it with `lym creel post --reply <id>`.
`internal/creelcli/cli.go:437` selects `VerbReply`, and `internal/creelengine/server.go:328` calls `Store.Reply`.
That method appends directly to the original sender at `internal/creelengine/verbs.go:400` without readiness checking.
The relay subsequently drops that mail when forwarding is off.
A stopped relay likewise produces no warning on this path.
Both correctness and affected-set lenses independently identified this defect; it is recorded once.

### [MEDIUM] Down-relay success does not carry the requested delivery field

**Location:** internal/creelengine/verbs.go:212

**Issue:** The board entry permits refusal with guidance or success carrying `delivered: false` and a reason.
The implementation instead stores the post and returns liveness plus a warning.
The CLI exposes `receiver_live` at `internal/creelcli/cli.go:472`, without the requested delivery field.
The warning communicates the queue state, but it does not implement either specified response contract.
The landing summary explains the writer's choice; no amendment to the decision record authorizes it.

### [LOW] The turn hook is a strand sender

**Location:** _lym/landing/summary.md:20

**Issue:** The summary groups the Slack turn hook with unchanged program senders.
`internal/slackcli/hook.go:173` obtains `orchClient`, which calls `newStrandClient` at line 213.
`internal/slackcli/wiring.go:131` wires that seam to `creelcli.NewStrandClient`.
The hook posts to the relay at `internal/slackcli/hook.go:192` and therefore reaches the new check.
The summary's exemption claim is inaccurate.

## Focus checks

The help removals compress still-valid details to fit the CLI size cap; package documentation retains them.
They do not change idle wake, the reply mark or parent resolution.
EXTERNAL is refused by orch classification; a beam thread normalizes to its beam's button owner.
A malformed sender cannot pass orch classification and then silently normalize to the prime.
The receiver comparison uses the actual relay address; differently spelled names are distinct receivers, not aliases.
The formatting-error bypass requires invalid internal hub geometry, not a supported alternative relay spelling.
Direct-post refusal precedes interrupt, journaling and waiter registration.
A relay is no strand, so an interrupting relay post has no successful interrupt path losing liveness.
Ordinary and waiting responses carry liveness, and waiting progress carries the warning to CLI stderr.
Program and absent-bridge bypasses have explicit test rows.
The fake's existing cross-machine refusal callers retain their meaning with the shared map.
No redundant-test finding is supported by the supplied reports or the distinct assertions.

## Validation

Reviewed the committed range from `5823c67e3bb8bc2e9aab7aed301ffcb08a3f424c` through HEAD.
Read the board entry through `lym board get` and compared it with the decision record.
Saved all four lens reports verbatim.
Attempted `lym gate test --tier 0 ./internal/creelengine ./internal/creelcli` with a one-hour shell timeout.
It exited before testing because the gate holder record could not be written under the read-only Bolt gate directory.
No fresh test pass is claimed; the supplied reach and redundancy reports remain the available gate evidence.
