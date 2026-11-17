---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: internal/creelengine/crossmachine.go:177
    summary: >-
      Code outside internal/agentname splits the machine address with strings.Cut on MachineAddressSeparator,
      at crossmachine.go:177 and machinemail.go:232, against PATTERN-agent-addressing.
    fix: >-
      Remove both splits and route them through internal/agentname.
      In replyAcrossMachinesLocked (crossmachine.go:177) replace strings.Cut(original.From, agentname.MachineAddressSeparator)
      with agentname.ParseMachineAddress(original.From, nil, machinename.CheckName) and use address.Name.String() as the receiving name.
      In machineHoldWayForward (machinemail.go:232) stop cutting the to argument.
      Have noticeReturnedLocked in outbound.go pass the receiving machine, parsed once with agentname.ParseMachineAddress,
      or add an exported agentname helper that returns the machine and name parts and call it from both sites.
      After the change no non-test file outside internal/agentname may hold strings.Cut or strings.Split on MachineAddressSeparator.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/slackengine/bridgein.go:168
    summary: >-
      An approve typed under a held peer message does not deliver the held message in that pass,
      so an approve then a revoke in one read leave the message held and undelivered, against D8.
    fix: >-
      In bridgeReader.readChannel, when handleOperator has carried out an operator command while a message is held,
      end the pass and run it again from the persisted delivery cursor, so the held message is tried again before the next operator message is handled.
      The operator cursor already keeps a handled command from running twice, and the rerun delivers the held messages in order and moves the delivery cursor past each one.
      Bound the reruns by the number of operator messages in the read.
      Extend the bridgein_test.go row that handles approve, revoke and plain text under a held message
      so that the message is delivered once after the approve and the revoke stands.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/slackengine/bridgeout.go:99
    summary: >-
      The relay settles the accepted posts of its record only at its start,
      so an entry whose settle was lost and whose message creel then returned at its hour stays until the next restart.
    fix: >-
      Call s.settleAccepted() at the top of bridgeSender.send before TakeMachine, and append its error to the failures that send returns.
      Keep the per-message branch for an accepted entry in the take loop, which still keeps a message whose settle failed from being posted again.
      Add a bridgeout_test.go row where a settle fails once, creel returns the message at its hour, and the next send settles the entry
      and drops it from the post record.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/creelengine/machinemail.go:231
    summary: >-
      machineHoldWayForward has no case for bridge-unsettled, so the return notice of a message the relay never held
      points at a relay notice that does not exist.
    fix: >-
      Add a case for MachineUnsettled to machineHoldWayForward that names what clears it,
      such as asking the operator to turn a slack button of this hub on so the relay runs, with the holdsAgain suffix.
      Assert that wording in the machinemail_test.go row that checks notices[1] for MachineUnsettled.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/slackengine/bridgechannel.go:46
    summary: >-
      When a create answers name_taken and the channel listing does not show the channel,
      ensureBridge caches nothing, so the paged FindChannel listing repeats at every bridge poll against D4's once per ceiling.
    fix: >-
      In channelKeeper.ensureBridge record the time of an attempt that ended in the not-listed error,
      and while bridgePollCeiling has not passed since it, return the same not-listed error without calling CreateChannel or FindChannel.
      Clear the record once an attempt succeeds or ends in a hold.
      Add a keeper_test.go row under the injected clock that checks FindChannel runs once per ceiling for that error.
  - id: F6
    severity: LOW
    class: consistency
    origin: handler
    location: internal/creelengine/crossmachine.go:46
    summary: >-
      maxMachineNameLength repeats the machine name rule's limit and machineEnvelopeLead repeats the envelope prefix of bridgemessage.go,
      so a change to either rule leaves a copy to drift, against PATTERN-code-conventions.
    fix: >-
      Export the limit from internal/machinename as MaxNameLength and use it in crossmachine.go in place of maxMachineNameLength.
      Export creelengine.MachineEnvelopeLead in place of machineEnvelopeLead,
      and define bridgeEnvelopePrefix in internal/slackengine/bridgemessage.go as that constant, as SlackProgram is defined as creelengine.BridgeProgram.
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/creelengine/machinemail_test.go:91
    summary: >-
      No test pins that a post to another machine starts no per-hub outbound worker, at the scheduler's start or on a post, as D3's criterion asks.
    fix: >-
      Add a row, or an assertion in an existing row of TestStore_MachineOutbound, that starts the scheduler with a hub directory fake
      which counts hub starts, posts to a machine address, and checks that no hub start was requested.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/hubgeom/hubgeom_test.go#TestSlackMachine
    summary: >-
      The redundancy report lists TestSlackMachine as contained, covered by TestSlackGeometry,
      and its only unique pin is the literal presence file name, which the covering test reads through a constant.
    fix: >-
      Add a line directly above func TestSlackMachine reading
      //testtiming:keep pins the literal slack-presence.json file name of SlackMachine, which TestSlackGeometry reads only through slackengine.PresenceFileName
---

### [BLOCKING] Code outside internal/agentname splits the machine address

**Location:** internal/creelengine/crossmachine.go:177, internal/creelengine/machinemail.go:232
**Issue:** `PATTERN-agent-addressing` and the decision record's Constraints allow only `internal/agentname` to form, parse or match the `<machine>::` name.
`replyAcrossMachinesLocked` cuts `original.From` and `machineHoldWayForward` cuts `to`, each with `strings.Cut` on `agentname.MachineAddressSeparator`.
A repository search finds no other non-test split outside `internal/agentname`, so both sites are the whole violation.

### [MEDIUM] An approve does not deliver the held message in the same pass

**Location:** internal/slackengine/bridgein.go:168
**Issue:** D8 says an approval delivers the held messages in order and moves the delivery cursor past each one.
`readChannel` skips every later non-operator message once one is held, and retries the held one only on the next poll.
With a held message followed by `approve` and `revoke` in one read, both commands run, the pin is gone before the held message is tried again, and it is never delivered.
The acceptance criterion about no second notice holds only because the notice mark survives.

### [LOW] Accepted posts are settled only at relay start

**Location:** internal/slackengine/bridgeout.go:99
**Issue:** D3 has the relay settle every accepted post of its record before it takes new mail.
`settleAccepted` runs once at start, and `send` settles only entries that `take` still returns.
An entry whose message creel returned at its hour stays in the record, and creel's late-delivery notice waits for a restart.

### [LOW] No way forward for bridge-unsettled

**Location:** internal/creelengine/machinemail.go:231
**Issue:** The return notice of a message the relay never reported a hold for falls to the default case, which points the orch at a relay notice that does not exist.

### [LOW] A create that finds no listed channel repeats the listing every poll

**Location:** internal/slackengine/bridgechannel.go:46
**Issue:** `ensureBridge` caches only hold reasons.
The "taken but not listed" error leaves nothing cached, so each poll repeats the create and the paged `FindChannel` listing, where D4 allows one listing per bridge-poll ceiling.

### [LOW] Rule constants repeated across packages

**Location:** internal/creelengine/crossmachine.go:46
**Issue:** `maxMachineNameLength` copies `machinename`'s unexported limit, and `machineEnvelopeLead` copies `bridgeEnvelopePrefix`.
Card 7's test pins the envelope width and not the prefix, so a change to either value drifts without a failing test.

### [LOW] No test for the absence of a per-hub worker on a post

**Location:** internal/creelengine/machinemail_test.go:91
**Issue:** D3's acceptance criterion names the scheduler's start and a post.
Only the reply row's title claims it starts no hub, so a `wakeOutboundLocked` call added to `postMachine` fails no row.

### [LOW] TestSlackMachine is contained and needs a keep mark

**Location:** internal/hubgeom/hubgeom_test.go#TestSlackMachine
**Issue:** The report lists it as contained, covered by `TestSlackGeometry`, which asserts the same `PeersPath` and the other machine paths through constants.
The one thing it pins that the covering test cannot see is the literal `slack-presence.json` name, so it is kept and marked rather than folded.

## Rejected

- code-tests, carry and retention have no test: `internal/creelengine/journal_test.go:974` asserts the carried machine-outbound entry with its receiving machine and last hold reason, and the carried machine-inbound entry.
- code-tests, no test for wait outcomes of machine mail: `machinemail_test.go:152` and `:195` pin the returned outcome of a wait, and `:303` calls `WaitReply` for a machine-outbound id.
- code-tests, return-notice wording per reason is not asserted: `machinemail_test.go:163` asserts the `approve osl` wording and the `bridge-unsettled` reason, and the wording of the other reasons is static prose.
- code-tests, the `daemonwakeups_test.go` list entry: it is a source list for the wakeup scan and no assertion on prose.
- code-scope, help text of `lym creel post` rewritten: `cmd/lym/clitree_test.go:280` caps `Long` at `maxLongBytes`, so the shortened lines are forced by the added paragraph the decision asks for.
- Declared PATTERN change: `PATTERN.md` and `pattern/PATTERN-agent-addressing.md` carry the declared text, so nothing is raised.
- Commit messages carry no `Pattern-Departure:` trailer, so no departure needs judging.
