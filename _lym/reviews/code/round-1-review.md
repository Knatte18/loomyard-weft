---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/reedengine/bar.go:591
    summary: The RC button is drawn in a session whose @lym_button_dir is empty, where its click cds to the empty word, fails in the orch wiring before RunE and shows no refusal
    fix: In internal/reedengine/bar.go, make remoteControlShownCondition also require the session option buttonDirOption, for example "#{&&:#{@strand},#{&&:#{" + remoteControlClickCommandOption + "},#{" + buttonDirOption + "}}}", combined with F3's change. Update the condition's comment, the RC lines of bar.go's file header and of internal/reedengine/doc.go, which say the button is drawn while the pane carries @strand and the click line is pinned. Update rcShownWhen in TestStatusFormatActions_TogglesFollowTheFollowButton (internal/reedengine/bar_test.go) to the new condition.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/orchcli/remotecontrol.go:159
    summary: The rc-on verb's new wiring has no test, neither a ClientAgentPane failure reported through output.Err nor the mark landing on the clicked pane's id
    fix: Add a test in internal/orchcli/remotecontrol_test.go that runs the rc-on verb through the cobra command with --tmux naming a fake tmux script under t.TempDir() that answers display-message with a pane line and records set-option calls. Assert the set-option -p -t <pane id> @lym_rc 1 call for an agent pane, and an error exit naming the client when display-message fails. Skip the orch wiring the same way the existing verb tests do, or call the RunE body through a seam if the wiring cannot run without a hub.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/reedengine/bar.go:591
    summary: The RC button is keyed on @strand, which a strand with an empty name still carries, while the click reads @lym_strand_name and refuses such a pane, contradicting the condition's own comment
    fix: In internal/reedengine/bar.go, key remoteControlShownCondition on "#{" + strandNamePaneOption + "}" in place of "#{@strand}", together with F1's change, and update the RC wording in bar.go's header, internal/reedengine/doc.go and rcShownWhen in internal/reedengine/bar_test.go to name @lym_strand_name.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/reedengine/paneoptions.go:4
    summary: paneoptions.go's file header and the strandNamePaneOption doc still say only the button click reads @lym_strand_name, but the RC click now reads it too
    fix: In internal/reedengine/paneoptions.go, change the header's sentence at line 4 to say no Go decision reads them back but the button click, which reads @lym_strand_name for {strand}, and the RC click, which reads it to name the agent it acts on; change the strandNamePaneOption comment at line 28 the same way.
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/orchengine/template.yaml:11
    summary: The remote_control and remote_control_name comments still describe an RC button on the orch session's bar and a name formed from the orch strand alone
    fix: In internal/orchengine/template.yaml, change remote_control's comment to say the RC button of an agent's pane turns remote control on for that agent, and change remote_control_name's comment so {role} and {name} are the role and full name of the strand the name is formed for, the orch's at launch and the clicked agent's for the RC button.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/slackcli/toggle.go:52
    summary: The slack bar --dir flag help still says run directory of the session, but reed now passes the session's orch directory, empty for a task
    fix: In internal/slackcli/toggle.go line 52, change the --dir usage text to name the directory of the orch whose slack button the bar shows, empty for a session with no such orch.
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/orchcli/remotecontrol.go:94
    summary: No test pins that a failed launch mark is logged and never fails the launch
    fix: In TestStart_LaunchPassesAndMarksRemoteControl (internal/orchcli/remotecontrol_test.go), add a row whose markRemoteControl fake returns an error with the switch on, and assert the launch still succeeds with the same spec and that the fake was called with the new guid.
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/reedengine/remotecontrol_test.go:742
    summary: TestMarkPaneRemoteControlOnVia's SetsThePaneOption subtest asserts the same set-option argv that TestMarkRemoteControlOn's MarksTheStrandsPane row asserts
    fix: Delete the SetsThePaneOption subtest from TestMarkPaneRemoteControlOnVia in internal/reedengine/remotecontrol_test.go and keep ATmuxFailureIsReturned, which pins the error naming the pane; fold the remaining subtest's body into the test function directly.
---

### [MEDIUM] RC button drawn where its click cannot run

**Location:** `internal/reedengine/bar.go:591`, with `internal/reedengine/panebin.go:290` and `internal/orchcli/cli.go` PersistentPreRunE.

**Issue:** `remoteControlShownCondition` requires only `@strand` and the pinned click line.
`@lym_button_dir` is empty whenever `composeButtonRowLocked` renders no buttons: a standalone geometry, a failed button context, an unresolvable shell, a failed beam lookup, or a place group with no entries.
The click line then runs `cd ` with an empty word, which lands in `$HOME` under posix, and `lym orch rc-on` fails in the orch wiring before RunE.
`ShowToggleOutcome` never runs and the line discards output, so the click does nothing visible.
Before the diff, `rc-bar --dir ""` printed nothing there and the button was not drawn, so this is a regression the new condition introduces.
The handler reached the same trace independently.

### [LOW] rc-on verb wiring untested

**Location:** `internal/orchcli/remotecontrol.go:159`

**Issue:** `TestTurnOnRemoteControl` covers the decision function, but the new RunE body is uncovered: a `ClientAgentPane` failure going through `ShowToggleOutcome` and `output.Err`, and the mark closure targeting `pane.PaneID`.
The lens filed it MEDIUM; the decision logic it wires is covered and the uncovered part is glue, so it is kept at LOW.

### [LOW] RC button drawn for an unnamed strand

**Location:** `internal/reedengine/bar.go:591`

**Issue:** `setStrandPaneOptionsLocked` (`internal/reedengine/paneoptions.go:36-45`) always sets `@strand` and unsets `@lym_strand_name` for an empty name.
The click reads `@lym_strand_name` and refuses with "runs no agent", while the condition's comment says a button no click could serve is never drawn.

### [LOW] paneoptions.go docs miss the RC click's read

**Location:** `internal/reedengine/paneoptions.go:4`, `:28`

**Issue:** Both comments say only the button click reads `@lym_strand_name`; `clientAgentPaneVia` (`internal/reedengine/remotecontrol.go:37`) now reads it too, as the updated `reedengine/doc.go` states.

### [LOW] orch template comments describe an orch-only RC button

**Location:** `internal/orchengine/template.yaml:11`, `:12`

**Issue:** The comments name "the RC button on the orch session's status bar" and "the orch strand's role" and "its full name".
The button now serves the agent of the active pane, and `turnOnRemoteControl` forms the name from that agent's strand name.

### [LOW] slack bar --dir help is stale

**Location:** `internal/slackcli/toggle.go:52`

**Issue:** The flag help still reads "run directory of the session whose bar is drawn"; the state line now passes `#{@lym_orch_dir}`, the orch's directory and empty for a task.

### [LOW] Failed launch mark branch untested

**Location:** `internal/orchcli/remotecontrol.go:94`

**Issue:** Every `markRemoteControl` fake in the launch and live-strand tests returns nil, so the log-and-continue branch the doc comment promises has no row.

### [LOW] Duplicate set-option assertion

**Location:** `internal/reedengine/remotecontrol_test.go:742`

**Issue:** `TestMarkPaneRemoteControlOnVia/SetsThePaneOption` asserts `set-option -p -t %7 @lym_rc 1`, which `TestMarkRemoteControlOn/MarksTheStrandsPane` (line 781) asserts through the public engine op.
Its sibling `ATmuxFailureIsReturned` pins the error message and stays.

## Focus verdicts

- Slack only in a prime's or a beam's session: holds. `composeButtonRowLocked` sets the orch directory only when the place is not `ButtonPlaceTask`, the task pins `''`, and `barSlackWord("")` returns no slack word. bar.go's header now matches what is drawn.
- RC per active-pane agent, never the parent orch: holds. The click reads the clicking client's active pane and injects into that pane's strand name through creel's remote-control command; the run directory only satisfies the orch wiring.
- RC off until clicked, `orch.yaml` unchanged: holds. Only a launch with `remote_control` on marks the orch's pane, adopting a running strand marks nothing, and the config switch and template values are untouched. Unresolvable agent and queue failure are refused and reported; the empty run directory path is not (F1).
- Removed state pin and helpers: no reference is left in code, docs or doc.go files; a repo-wide grep for every removed symbol, option and file name finds nothing.
- set-option counts follow from the removed `@lym_rc_state_command` pin; test economy is F7 and F8.

## Rejected

- The away toggle still draws in a task session: not a defect. The amended entry scopes "Slack stays per orch" to the per-orch state, and the away toggle switches the machine's presence, which no parent orch owns.
- The click marks an agent on an engine with no remote control: not a defect. The entry asks for the same creel command for every agent, the pane carries no engine, and the change description and orchcli doc state the behaviour.
