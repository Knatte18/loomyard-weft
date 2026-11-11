---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/slackengine/relay.go:625
    summary: "The relay never ends itself when retire drops the last button, so it keeps the machine connection with no active orch"
    fix: "In wakeLoop.wake in internal/slackengine/relay.go, after a successful ReadForwarding and the ensureForwardedChannels call and the retire call, set w.exiting = true when the list read is empty, so that wakeLoop.run returns nil and RunRelay ends. Update the relay tests in internal/slackengine/relay_test.go that start a relay with no button on, and add one TestRunRelay row where retire drops the only button and the relay returns."
    origin: lens:code-correctness
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/slackengine/keeper.go:411
    summary: A forwarding entry of a beam that was pressed on but has no channel entry is never dropped, so the relay is started and kept for an orch that is gone
    fix: "In channelKeeper.retire in internal/slackengine/keeper.go, besides the channel-map loop, read ReadForwarding, and for each forwarded beam orch whose slug a successful beams read does not list and whose channel map entry is absent or already archived, call dropForwarding. Do it only when the beams read stood, as retire already requires. Add a row to the Retire subtests in internal/slackengine/keeper_test.go with a pressed beam that has no map entry."
    origin: lens:code-correctness
  - id: F3
    severity: MEDIUM
    class: decision
    location: internal/boardengine/render.go:185
    summary: The diff adds an exported RootProjectHandle to the board module, which the Limits list of the decision record does not license, and nothing tests it
    fix: "Leave the code in place and defer this finding to the run's parent: the only sound change is recording or refusing the correction with lym loom decision add, which lies outside the fixer's write surface. When the parent records it, cover boardengine.RootProjectHandle through the test of orchProject named in F4."
    origin: handler
  - id: F4
    severity: MEDIUM
    class: design
    location: internal/slackcli/wiring.go:178
    summary: worktreeOrch and orchProject decide which worktrees get a button and the project in the channel name, and no test reaches either, since the harness fakes the worktreeOrch seam
    fix: "Add a table-driven test in internal/slackcli beside wiring.go that calls worktreeOrch for the prime, a listed beam, a pair's worktree and an unlisted worktree, and orchProject for the empty slug, a beam with a project entry, a beam with none and an unreadable board, using a temporary hub fixture through hubforge."
    origin: lens:code-tests
  - id: F5
    severity: LOW
    class: design
    location: internal/slackengine/keeper.go:426
    summary: When dropForwarding fails after the archive stands, the entry is archived and skipped by every later retire, so the button is never dropped
    fix: "In channelKeeper.retire, call dropForwarding for an archived entry whose orch is not listed in beams, not only after a successful retireEntry, so a failed drop is retried at the next retire pass. This is the same pass F2 adds, so make both in one change."
    origin: lens:code-correctness
  - id: F6
    severity: LOW
    class: consistency
    location: pattern/PATTERN-daemon-wakeups.md:93
    summary: The Slack relay bullet still names the inbox and presence watch, which the diff changed to the inbox and the forwarding file
    fix: "In pattern/PATTERN-daemon-wakeups.md line 93 replace 'the inbox and presence watch' with 'the inbox and forwarding-file watch'."
    origin: lens:code-affected-set
  - id: F7
    severity: LOW
    class: design
    location: internal/slackengine/relay.go:597
    summary: The diskCheck fallback that compares the forwarding list when the watch cannot open has no test row
    fix: "Add a TestRunRelay subtest in internal/slackengine/relay_test.go whose Watch seam returns an error, presses a button, advances the clock and expects the orch's channel to be made."
    origin: lens:code-tests
  - id: F8
    severity: LOW
    class: design
    location: internal/slackcli/hook.go:177
    summary: No TestHook row posts for a beam orch whose own button is on, so the orch name derived with TaskSlug is unpinned
    fix: "Add a TestHook row in internal/slackcli/hook_test.go with a beam orch strand, such as hub:b:orch, its own button on, presence slack, expecting action forwarded and a post to hub:SLACK."
    origin: lens:code-tests
  - id: F9
    severity: LOW
    class: design
    location: internal/slackengine/inbound.go:614
    summary: The branch where reading the orch's button fails and the message goes to retryOrGiveUp has no test row
    fix: "Add a row to the inbound handling subtests in internal/slackengine/inbound_test.go that writes an undecodable forwarding file, says a message, and expects the cursor to stay before it and no post."
    origin: lens:code-tests
---

### [MEDIUM] The relay never ends itself when retire drops the last button

**Location:** internal/slackengine/relay.go:625, internal/slackengine/keeper.go:426
**Issue:** The decision record says the relay runs only while at least one orch is active.
Only the toggle verb calls the relay's stop, when the last button it flips goes off.
Retire drops a retired beam's button from the forwarding file, and nothing then ends the relay.
`ensure` refuses to start one with no button, but a live relay keeps its Socket Mode connection until a press or `lym slack stop`.

### [MEDIUM] A forwarding entry of a beam with no channel entry is never dropped

**Location:** internal/slackengine/keeper.go:411
**Issue:** Retire walks the channel map only, so a beam whose button was pressed on, and whose channel was never made, stays in the forwarding file after the beam retires.
`ReadForwarding` stays non-empty, so `start.ensure` keeps starting the relay and `forwardingOff` of another orch never ends it.
A later beam of the same slug also starts with its button already on.

### [MEDIUM] RootProjectHandle is board-module code outside the Limits list

**Location:** internal/boardengine/render.go:185
**Issue:** The record's Limits name only slackengine, slackcli, the reed Slack toggles, the orch role file and their tests.
The diff adds an exported function to the board module so the channel name can carry the root project's handle `p0`.
The change is small and a named constant is what PATTERN-code-conventions asks for, but no record entry licenses it, and no test reaches it.
It is an unrecorded departure from the amended entry.

### [MEDIUM] worktreeOrch and orchProject are untested

**Location:** internal/slackcli/wiring.go:178
**Issue:** "A pair's orch gets no button" and the project in the channel name are decisions of the record.
Both live in `worktreeOrch` and `orchProject`, which the harness replaces with a fake or never calls, so a regression in either passes every test.

### [LOW] A failed button drop after an archive is never retried

**Location:** internal/slackengine/keeper.go:426
**Issue:** `retire` skips an entry once it is archived, so when `dropForwarding` fails after the archive stands, the button stays on for good.

### [LOW] PATTERN-daemon-wakeups still names the presence watch

**Location:** pattern/PATTERN-daemon-wakeups.md:93
**Issue:** The relay's wake watch covers the inbox and the forwarding file now, and a write to the presence file wakes nothing.

### [LOW] The diskCheck forwarding comparison is untested

**Location:** internal/slackengine/relay.go:597
**Issue:** With the watch closed, the comparison is the only way a button press wakes the relay, and no test row exercises it.

### [LOW] No hook test for a beam orch with its button on

**Location:** internal/slackcli/hook.go:177
**Issue:** The only beam row uses another orch's button, so the orch name `TaskSlug` derives for a beam strand is not pinned by a post.

### [LOW] The unreadable-button branch of inbound handling is untested

**Location:** internal/slackengine/inbound.go:614
**Issue:** A failed read of the forwarding file in `handle` holds the message through `retryOrGiveUp`, a new failure path with no row.

## Rejected

- Scope finding on `internal/hubgeom/slackgeom.go`: it wires the `ForwardingPath` of the changed `Geometry` and drops `SwitchPath`, a forced consequence of the slackengine change, and not a separate change.
- Scope finding on `docs/overview.md`: the repository's rules require the doc of a changed module to land in the same diff.
- No pin that the literal `orch-channels.json` differs from the old name: the tests use the constant, and the rename is the way the record's "leave existing channels alone" is met; a test of the literal would pin a constant's value.
- A legacy entry under a dated name without the `Renamed` flag is renamed again: the old map file is never read, so no such entry exists.
- A button toggled off between another press's write and its relay start could end the relay under a lit button: the window is two file writes wide, and the next prime bring-up starts the relay.
- No redundant or duplicated test found among those the redundancy report lists as not judged; each pins a behavior the diff changed.
