```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens. It traces each change in the diff to a goal point of the decision record and checks that every goal point has a change that builds it.

**Decisions.** The answer key is the decision record alone. The record has no `Added after Discussion` entries, so the goal points are:
- **G1:** `switchClientVia` reads the clicking client's own `#{session_id}`, not the most recently active client's.
- **G2:** `runButtonVia`'s `{strand}` lookup reads `#{@lym_strand_name}` from the clicking client's own session.
- **G3:** a test of G1 and G2 uses the two-client setup that `TestClientAgentPane_ReadsEachClientsOwnSession` shows.
- **G4:** a shellguard row refuses `systemd-run`, `busctl --user`, `dbus-send --session` and `gdbus` in run sessions.
- **G5:** a shellguard row refuses `tmux -S` or `-L` aimed at another hub's server.
- **G6:** lym's own use of the bus keeps working, because the rows apply to agent commands only.

**Changes and what each traces to.**
- C1, `clientrow.go` `readClientFormatsVia`: G1 and G2.
- C2, `switch.go` reads through C1 and updates its doc: G1.
- C3, `buttonclick.go` `buttonStrand` reads through C1 and updates its doc: G2.
- C4, `remotecontrol.go` `clientAgentPaneVia` moves onto C1: forced by C1, since the three readers share one implementation.
- C5, `switch_test.go` and `buttonclick_test.go` rows move to `list-clients`: forced by C2 and C3.
- C6, `switch_integration_test.go` `TestSwitchClient_StepsFromTheClientsOwnSession` and `buttonclick_integration_test.go` `TestRunButton_StrandNamesTheClickingClientsOwnPane`: G3.
- C7, `remotecontrol_integration_test.go` extracts `attachOneClientPerSession`: forced by C6, which shares the attach helper.
- C8, `reedengine/doc.go` gains the `display-message -c` tmux fact: forced doc by C1–C3.
- C9, `rows.go` `session-bus` row, its steer, notice, pattern and `Forms`: G4.
- C10, `rows.go` `hub-tmux-server` row, its prefix constant, steer, notice and pattern: G5.
- C11, `rows.go` `machine-sleep` comment drops `busctl` and `dbus-send` from the routes that pass: forced by C9, whose row now refuses them.
- C12, `shellguard/doc.go` paragraph on the two rows: forced doc by C9 and C10.
- C13, `rows_integration_test.go` rows, plus the `reedengine.ServerName` drift check: forced tests of C9 and C10.
- C14, `claudeengine/settings_test.go` counts and the `settings-golden.json` golden: forced by C9 and C10.
- G6 needs no code change: both rows are PreToolUse Bash patterns, and lym's Go-side bus use never meets them.

**Decisions and the changes that build them.**

| Decision | Built by |
|---|---|
| G1 | C1, C2 |
| G2 | C1, C3 |
| G3 | C6 |
| G4 | C9 |
| G5 | C10 |
| G6 | the rows' PreToolUse placement in C9 and C10 |

No decision is left unbuilt, and no change traces to nothing.

**Findings**

1. **LOW, class `scope`.**
   - **Location:** `internal/shellguard/rows.go`, `sessionBusCommandPattern` (diff piece 002, `rows.go` hunk at line 596 of the diff output).
   - **Summary:** the row refuses `dbus-send` with any flags, `dbus-send --system` included. The answer key asks to refuse `dbus-send --session`.
     - The default form, with no bus flag, reaches the session bus, so refusing it is within the entry's intent.
     - Refusing an explicit `--system` call goes past what the entry licenses, and no `Pattern-Departure:` trailer or added entry covers it.
   - **Evidence:**
     - The decision record's `## Goal` names "`dbus-send --session`".
     - The pattern in `rows.go` is `(systemd-run|dbus-send|gdbus)` + `wordEndPattern`, with no exclusion for `--system`.
     - The summary's "Design decisions" puts it as "refused whatever its flags".
   - **Fix:** exclude a `dbus-send` whose arguments carry `--system`, and add a row for it to `TestRows_PatternsDenyAndAllow`. Otherwise the parent records the widening with `lym loom decision add`.

**Rejected candidates**

- **`hub-tmux-server` also refuses the session's own hub server, not only "another hub's".** Not a scope finding.
  - A static pattern cannot tell the two apart.
  - The only sessions that reach this row with raw tmux allowed are sampler sessions, which the entry wants kept off hub servers.
  - The entry's purpose, keeping agents off hub servers, holds, and the summary's design decisions state the call.
- **Both rows are installed in the orch as well (`SwitchNone`).** Not a scope finding. The entry says "run sessions", and the orch is a run session under the same "every run, the orch included" convention its sibling rows follow.

Outside this lens: `_lym/landing/summary.md` says `clientAgentPaneVia`'s "argv and errors are unchanged", but the not-attached and unreadable-row errors now carry an extra "read the active pane of client %q:" wrapper. That claim belongs to the handler.

Tool calls: none.
