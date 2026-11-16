---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/session.go:1211
    summary: Runner.TurnOnRemoteControl has no test; its guarded paste, multi-line refusal, no-capability no-op and toldErr short-circuit are unpinned
    fix: In internal/shuttleengine/session_test.go add a TurnOnRemoteControl case beside the peers that checkPastedSessionCommand covers, asserting the pasted text is "/remote-control <name>" followed by Enter; add a row to the engine dispatch table in internal/shuttleengine/enginedispatch_test.go asserting an engine without RemoteController types nothing and returns nil; add a row asserting a name holding "\n" returns an error and types nothing.
  - id: F2
    severity: MEDIUM
    class: decision
    origin: handler
    location: internal/orchcli/remotecontrol.go:288
    summary: The RC click sends /remote-control into the orch pane even when the record reads on, and Claude Code's answer to the command in a session that already has it on is unverified
    fix: Verify what Claude Code shows when /remote-control is typed into a session whose remote control is already on. If it opens a panel or dialog with no input box, make turnOnRemoteControl refuse while orchengine.RemoteControlOn reads on for the live strand, with a way forward naming /remote-control typed by hand in the orch pane; otherwise state in the rc-on help and the orchcli doc's Remote control section that a click on a lit button is harmless, naming what Claude Code shows.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/orchcli/remotecontrol_test.go:79
    summary: The launch's unlisted-strand name, its name-formation failure and the resume branch passing the name have no test row
    fix: In TestStart_LaunchPassesAndRecordsRemoteControl add a row with no listed strand (expecting the name formed from FixedStrandName, such as neve::tst:orch), a row whose machineName returns an error (expecting an empty RemoteControlName, a successful launch and the record off), and a row seeding orch state with a SessionID (expecting the resume spec to carry the name).
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/orchcli/remotecontrol_test.go:179
    summary: An unreadable remote-control record reading as rc-off, and rc-bar printing nothing for an empty or unresolvable dir outside the orch wiring, have no test row
    fix: Add an UnreadableRecord row to TestRemoteControlBarWordAt that writes invalid JSON at paths.RemoteControlPath for a state with a strand and expects reedengine.RemoteControlUnlit; add one row running the orch command tree with `rc-bar --dir <t.TempDir()>` and expecting an empty line and exit 0.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/orchcli/remotecontrol.go:297
    summary: With two orch strands in the worktree the RC click refuses as if no live orch strand ran, naming lym beam start instead of the duplicate strands
    fix: In turnOnRemoteControl split the check - when len(named) > 1, refuse naming the duplicate strands' guids with the way forward `remove the extras with lym reed remove and click RC again`, as startOrch does at internal/orchcli/start.go:613; keep the current refusal for zero strands or a dead one.
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/orchcli/start.go:684
    summary: The spawn-watcher and already-running branches adopt a listed strand into the state without a remote-control record, so a live orch launched with the flag reads dim after an adopt
    fix: In startOrch's StartSpawnWatcher and default branches, after adoptListedStrand succeeds with st.Strand differing from strand.GUID, carry the record over - call c.recordRemoteControl(strand.GUID, on), where on is orchengine.RemoteControlOn(c.paths, st.Strand) read before the adopt.
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/orchcli/remotecontrol.go:308
    summary: When the inject is queued but the record write fails, rc-on reports a failed click although the command will still be typed
    fix: In turnOnRemoteControl, log a failed WriteRemoteControl at Warn after a successful inject and return the success fields, as recordRemoteControl does for the launch, instead of returning the write error.
  - id: F8
    severity: LOW
    class: scope
    origin: handler
    location: internal/orchcli/remotecontrol.go:366
    summary: A Codex orch still draws a dim RC button whose click only refuses, though the board entry gives a Codex orch nothing
    fix: Record the engine's lack of remote control so the bar draws no button - in startOrch write the record only when c.orchHasRemoteControl() holds, and have remoteControlBarWordAt return no word when no record exists for the strand; update TestRemoteControlBarWordAt's NoRecord row to expect an empty word, and the reed and orch docs that say the button shows while the worktree records an orch strand.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/orchcli/cli.go:148
    summary: The orch group's help still says every verb runs from the prime or a beam, though rc-bar now runs from any session's directory
    fix: Amend the orch group's Long text in internal/orchcli/cli.go to say every verb but the hidden rc-bar runs from the hub's prime worktree or from a beam.
  - id: F10
    severity: LOW
    class: design
    origin: handler
    location: internal/reedengine/bindings_test.go:23
    summary: The redundancy report lists TestBindingArgvs_BindsTheFiveAltKeysAndTheStatusClickInTheRootTableOnly and TestComposeSwitchCommand as contained by TestPinBindingsLocked_PinsTheToggleLinesOnlyWhenEverythingResolves, but both pin exact argv order and line text the covering test never asserts
    fix: Add `//testtiming:keep pins the exact bind-key argvs, branch order and separator count that the pin test only checks for presence` directly above `func TestBindingArgvs_BindsTheFiveAltKeysAndTheStatusClickInTheRootTableOnly` in internal/reedengine/bindings_test.go, and `//testtiming:keep pins the exact composed shell lines per dialect and the named warning of each unresolvable executable` directly above `func TestComposeSwitchCommand` in internal/reedengine/panebin_test.go.
---

The diff makes the change the board entry asks for.
Every orch launch and resume through `startOrch` passes `--remote-control` with the name.
The name comes from orch.yaml's template, which defaults to `{machine}::{name}`, the `agentname.FormatMachineAddress` form.
The switch is on in the template.
Only the orch spec sets the field, and a Codex default engine gets no flag.
The name is quoted once per dialect on both launch lines.
The RC button reaches the pane only through creel and shuttle's guarded send, and it is lit from the record lym last wrote, the fallback the entry allows.
The `orchroute` and creel changes each serve that path.
No PATTERN entry is broken, no departure trailer is present, and no PATTERN change is declared.
No finding is BLOCKING.

### [MEDIUM] Runner.TurnOnRemoteControl is untested

**Location:** internal/shuttleengine/session.go:1211

**Issue:** The method's peers (`ReloadPlugins`, `TypeColor`) are pinned through `checkPastedSessionCommand` and the engine dispatch table.
The new method's behaviours are not pinned: the paste, the refusal of a multi-line name, the nil sequence on an engine without `RemoteController`, and the `toldErr` return.
The only test that reaches it, `TestHubDelivery`, calls a recording fake.

### [MEDIUM] The RC click types into a session that may already have remote control on

**Location:** internal/orchcli/remotecontrol.go:288

**Issue:** The board entry has the button turn remote control on "in a session running without it".
The change description chose to send the command on every click, lit or not, so that a click recovers a session the operator turned off inside Claude Code.
What Claude Code does with `/remote-control` in a session that already has it on was not verified.
If it opens a panel with no input box, two things can follow.
The orch pane holds a modal that stops later creel deliveries until the operator dismisses it.
And, as the correctness lens notes for `sendVerified` with `startsNoTurn`, the send may read as not landed and be typed again.
The author must settle this against Claude Code's real behaviour.

### [LOW] Launch branches without test rows

**Location:** internal/orchcli/remotecontrol_test.go:79

**Issue:** `TestStart_LaunchPassesAndRecordsRemoteControl` seeds only a listed strand, a machine name that resolves, and a state with no SessionID.
So three paths have no row: the name formed through `FixedStrandName`, the Warn-and-launch-without fallback, and the resume spec carrying the name.

### [LOW] rc-bar's fallback paths without test rows

**Location:** internal/orchcli/remotecontrol_test.go:179

**Issue:** The rc-bar help promises two things that nothing pins.
A record it cannot read prints `rc-off`.
A directory that resolves to no worktree prints nothing, because the verb skips the orch wiring.

### [LOW] Duplicate orch strands misreported by the click

**Location:** internal/orchcli/remotecontrol.go:297

**Issue:** `len(named) != 1` folds two orch strands into "this worktree runs no live orch strand" with the way forward `lym beam start`.
`startOrch` refuses the same case naming the extras and `lym reed remove`.

### [LOW] Adopted strand loses its lit state

**Location:** internal/orchcli/start.go:684

**Issue:** `adoptListedStrand` rebinds `st.Strand` to a listed strand's guid without touching the remote-control record.
`remoteControlBarWordAt` then reads the record against the new guid and draws the button dim for an orch that may have been launched with the flag.

### [LOW] Record write failure reported as a failed click

**Location:** internal/orchcli/remotecontrol.go:308

**Issue:** The inject is already queued when `WriteRemoteControl` runs, so the command will be typed.
Its failure still surfaces as a refusal on the bar and an error envelope, and the button stays dim.

### [LOW] RC button drawn on a Codex orch

**Location:** internal/orchcli/remotecontrol.go:366

**Issue:** The board entry says a Codex orch gets nothing.
`remoteControlBarWordAt` shows the button for any recorded strand, so a Codex orch shows a dim RC whose click only refuses.
The change description lists this as an open risk, not as a choice the entry licenses.

### [LOW] Orch group help is stale

**Location:** internal/orchcli/cli.go:148

**Issue:** The group's Long text says every verb runs from the prime or a beam.
`rc-bar` is now exempt from that wiring at `internal/orchcli/cli.go:122`.

### [LOW] Contained reed tests need keep marks

**Location:** internal/reedengine/bindings_test.go:23

**Issue:** The redundancy report lists both tests as contained.
Their covering test builds its expected lines through the same compose functions and checks only that each branch is present.
So the exact argv order, the separator count and the per-dialect line text are pinned only by the contained tests.
Each should therefore carry a keep mark rather than be folded.

## Rejected

- Recording before `saveFreshLaunch` can leave the record naming a guid the state does not, if the save fails: display only, and a failed save already fails the bring-up, so the next start rewrites both.
- `orchHasRemoteControl` looking up an empty engine name: the orch spec never sets `Engine`, so `Lookup("")` resolves the same default engine `Start` uses.
- The lock `remote-control.json.lock` beside its file: the file is under `.lym`, not tracked, and sits like `state.json.lock`; PATTERN-lym-files' lock rule covers tracked files.
