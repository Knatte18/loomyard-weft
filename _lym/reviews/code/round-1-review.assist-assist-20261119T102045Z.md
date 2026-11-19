---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/creelengine/verbs.go:193
    summary: Replies to Slack bypass the relay check and can report success for mail the relay drops.
    fix: In Store.Reply, check relay eligibility and state when original.From is this hub's Slack program, before appending or settling any reply. Refuse a disabled button and a down relay with their state and way forward, or carry the requested undelivered result through VerbReply and the CLI reply branch. Extend the existing reply behavior tests with a Slack-origin message followed by a disabled button and a down relay, asserting the result and absence of mutation on refusal.
    origin: lens:code-correctness
  - id: F2
    severity: MEDIUM
    class: scope
    location: internal/creelengine/verbs.go:212
    summary: A down relay returns success without the delivered false field required by the goal.
    fix: Preserve the queued post and warning, but add an optional delivered field set to false for the down-relay result and carry it through PostResult, Response, waiting responses and postCmd output with its reason. Update the existing down-relay test assertion and affected documentation. Do not infer delivered true from a live relay. Alternatively, refuse the down-relay post before storage with its state and the slack start command, as the goal permits.
    origin: lens:code-scope
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/creelengine/machinemail_test.go:292
    summary: The new EXTERNAL-to-Slack refusal has no regression coverage.
    fix: Add an EXTERNAL sender row to TestStore_PostToTheSlackRelay with a refusing bridge and assert the propagated refusal and no journaled post. Add ly:EXTERNAL to TestSlackBridge_RelayPostReady and assert the non-orch refusal and its pane-or-orch way forward.
    origin: lens:code-tests
  - id: F4
    severity: LOW
    class: scope
    location: internal/creelcli/cli.go:379
    summary: The help edit drops unrelated descriptions of behavior that remains supported.
    fix: Restore concise descriptions in postCmd.Long of waking idle hook receivers, the expectation mark beside the reply command, and the resolved receiver in a parent post answer. Shorten surrounding wording to retain the existing help-size cap.
    origin: lens:code-scope
  - id: F5
    severity: LOW
    class: consistency
    location: _lym/landing/summary.md:19
    summary: The summary incorrectly identifies the Slack turn hook as an exempt program sender.
    fix: Remove the Slack turn hook from the examples of exempt program senders in the summary. State that it uses a strand client and reaches the new check; retain the program exemption description for actual program clients.
    origin: handler
---

### [MEDIUM] Slack replies still report silent success

**Location:** internal/creelengine/verbs.go:193

**Issue:** The new check runs only through `Store.post`.
The CLI's `post --reply` branch sends `VerbReply`, whose implementation appends directly in `Store.Reply`.
Slack inbound mail is posted from the relay program (`internal/slackengine/inbound.go:750`).
An orch can receive that mail, have its button turned off, and reply successfully into the relay mailbox.
The relay then drops the reply under its forwarding check (`internal/slackengine/outbound.go:201`).
The same reply path emits no relay-down warning after the relay stops.
This leaves a user-facing form of posting to Slack outside the requested reporting behavior.

### [MEDIUM] The down-relay response does not meet the goal's contract

**Location:** internal/creelengine/verbs.go:212

**Issue:** The board entry permits refusal or success with `delivered: false` and a reason.
The implementation instead sets `ReceiverLive` and `Warning`;
the CLI exposes `receiver_live` and prints the reason on stderr.
That warns a reader, but it omits the specified machine-readable delivery result.
The summary acknowledges the substitution, and the decision record contains no amendment authorizing it.
Relay liveness also cannot establish successful delivery, so adding a true delivery claim would be incorrect.

### [MEDIUM] EXTERNAL is absent from the new coverage

**Location:** internal/creelengine/machinemail_test.go:292

**Issue:** The store table covers strand and program senders, but not EXTERNAL.
The production bridge table likewise covers a non-orch strand without an EXTERNAL case.
Changing the engine guard to bypass all non-strand senders would leave both tables passing.
That would silently store EXTERNAL mail even though the new contract says it reaches the bridge and is refused.
The program and missing-bridge exemptions already have rows and need no duplicate tests.

### [LOW] Help loses existing behavior descriptions

**Location:** internal/creelcli/cli.go:379

**Issue:** The removed descriptions still hold in code.
`wakeHookStrand` rings idle receivers, `MessageLine` renders the expectation beside the reply command,
and the parent branch of `postCmd` still adds the resolved receiver to its output.
The help-size commit explains the pressure to shorten the text;
it does not require removing these meanings to document Slack reporting.

### [LOW] The summary misclassifies the turn hook

**Location:** _lym/landing/summary.md:19

**Issue:** The summary groups the Slack turn hook with program senders that ask the bridge nothing.
`internal/slackcli/wiring.go:130` constructs its client through `creelcli.NewStrandClient`.
That constructor claims the strand and no program (`internal/creelcli/goclient.go:75`).
The hook's post therefore reaches the new relay check, contrary to the summary's exemption claim.

## Checks and focus outcomes

Reviewed the supplied diff pieces and every commit message in
`5823c67e3bb8bc2e9aab7aed301ffcb08a3f424c..HEAD`, against board entry T8 and its decision record.
No declared PATTERN change or prior finding was supplied.

The normal post refusal precedes interruption, storage, superseding and waiter registration.
An interrupt cannot target the Slack program because it requires a listed strand.
Ordinary responses carry liveness and warning;
waiting posts send the warning as progress and the existing CLI callback prints it on stderr.
Existing response and CLI tests exercise those shared paths.

EXTERNAL and malformed sender names fail the orch predicate before slug normalization.
Orch threads normalize to their owning orch, and eligible beams use that orch's button.
The explicit refusal branches name a way forward.
No alternate spelling of the relay mailbox was found: scoped and lowercase names are distinct receivers.
No valid production geometry was found that makes FormatProgram fail and bypass the check.
The fake's existing cross-machine refusal callers do not enter the local relay branch.
Its shared refusal map therefore leaves those tests' meaning intact.
The changed package and result comments describe the implemented behavior, including its explicit exemptions.

The new tests use parallel isolated fixtures, a fixed timestamp and a held lock, without added sleeps.
The targeted foreground gate was attempted with a 3600-second command bound:
`lym gate test --wait --tier 0 ./internal/creelcli ./internal/creelengine ./internal/testkit/creelfake`.
Its arguments narrowed execution to the new bridge and store tests.
It could not acquire a slot because the sandbox made the holder-record directory under `_bolt/.lym/gate` read-only.
No test result is claimed from that attempt.
All lens reports were saved verbatim beside this report.
The verdict follows the format's severity rule: no finding is classified BLOCKING.
