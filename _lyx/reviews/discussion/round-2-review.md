---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D5, third bullet
    summary: >-
      D5 names lyx shuttle send as the only programmatic driver of the bypass session, but creel inject and the creel post reminder also type into a recordless strand's pane, so the stated reach and bound are wrong
    fix: >-
      In decision-record.md D5, rewrite the third bullet's second and third lines so the session takes no turn until text reaches its pane through any channel that types there, naming the operator in its window, `lyx shuttle send <guid>`, `lyx creel inject` and the reminder line a `lyx creel post` types; then state the bound each channel carries instead of saying only the explicit `--permission-mode bypass` bounds it, namely that send refuses under a covering pause, that creel lets EXTERNAL callers, programs and orch strands inject on any strand and any other strand on its own only, that any strand of the hub may post to it and so start a turn through the reminder, and that creel's sender identity boundary is the operator's user (the 0700 lyxtemp root).
  - id: F2
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D1, and D5 first and fourth bullets
    summary: >-
      The record says an empty mode makes the session prompt and that bypass removes only the dialog, but with no permission flag the session starts in the settings layers' permissions.defaultMode, which on this machine is auto
    fix: >-
      In decision-record.md D1, replace "Empty, the flag's default, keeps today's behaviour: the session prompts." with "Empty, the flag's default, keeps today's behaviour: the line carries no permission flag, so the session starts in the default mode its Claude Code settings set, prompting when none sets one." In D3, add after the mapping sentence that `prompt` adds nothing and so cannot override a settings default mode. In D5's first bullet, add that lyx turns bypass on only through the explicit flag, while a settings layer's `defaultMode` can still set the mode outside lyx, as today. In D5's fourth bullet, replace "bypass removes only the dialog that stood in for them" with "bypass replaces whatever default mode the settings set with skipping every dialog".
  - id: F3
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D5, second bullet
    summary: >-
      D5 cites the worktree settings' hooks as a bound, but buildOpenCmd installs no hooks and the worktree's .claude/settings.json holds tool deny rules and no hooks, so the hook half of the bound rests on settings lyx does not provide
    fix: >-
      In decision-record.md D5, rewrite the second bullet as "The session loads Claude Code's own settings layers, the worktree's included, and Claude Code applies their deny rules and hooks in bypass mode too (Claude Code behaviour, not lyx code); lyx installs no rule or hook of its own on an opened session."
---

### [MEDIUM] D5 understates who can drive the unwatched bypass session

**Location:** `_lyx/discussion/decision-record.md` D5, third bullet

**Issue:**
D5 says the session takes no turn until text is typed by the operator or sent by `lyx shuttle send <guid>`, and that nothing but the explicit flag bounds a `send`-driven session.
The `send` half holds: `Runner.Send` (`internal/shuttleengine/run.go:1130`) falls back to `liveStrand` on `ErrNoRunRecord` and types through `sendVerified`, and `internal/shuttlecli/send.go` refuses under a covering pause first.
`Runner.Open` (`internal/shuttleengine/open.go`) adds the strand with `render.AnchorOwnWindow` and returns without waiting, so nothing watches it by construction.
But send is not the only automated channel.
`internal/creelengine/doc.go` states its delivery works on a strand with no run record, such as a hand-added session, through shuttle's verified send.
`lyx creel inject` (`internal/creelcli/cli.go`) types a line into any strand for EXTERNAL callers, programs and orch strands, and `lyx creel post` lets any strand leave a message whose reminder line is typed into the idle receiver and starts a turn.
So any strand of the hub can steer a bypass session through creel, and the bounds that apply are creel's sender rules and identity boundary, which D5 omits.
This is the attack-surface question the rubric requires answered: the routing edge's reach is stated short of what the code allows.

### [LOW] An empty or `prompt` mode does not make the session prompt

**Location:** `_lyx/discussion/decision-record.md` D1, D5 first and fourth bullets

**Issue:**
`buildOpenCmd` (`internal/shuttleengine/claudeengine/command.go:248`) passes no permission flag, and `validatePermissionMode` maps both empty-interactive and `prompt` to adding nothing.
Claude Code then starts the session in the `permissions.defaultMode` its settings layers set; `~/.claude/settings.json` on this machine sets `"defaultMode": "auto"`, so an opened session does not prompt here today.
D1's "the session prompts", D5's "nothing but an explicit `--permission-mode bypass` turns it on" and "bypass removes only the dialog that stood in for them" rest on that false premise.
Behaviour stays unchanged, which keeps this LOW, but the record's guarantee for `prompt` is one the code cannot give.

### [LOW] The hook bound names hooks lyx does not install

**Location:** `_lyx/discussion/decision-record.md` D5, second bullet

**Issue:**
The bullet bounds the bypass session by the worktree settings' deny rules and hooks.
The open line carries no `--settings`, so lyx installs nothing; the worktree's `.claude/settings.json` holds `permissions.deny` tool entries and no `hooks` key.
Hooks reach the session only from other Claude Code settings layers the operator controls.
The deny-rule half is accurate; the hook half names a bound the code does not provide, which is what the focus asks to settle.

## Rejected

- D3's refusal order: `Runner.Open` checks the engine, then the covering pause, then `SessionOpener`, then calls `OpenLaunch`, which loads `claude.yaml` and validates effort and model before minting a session id; a mode check placed there reports after all three, as D3 says, and before the marker directory and `AddStrand`, as AC4 says.
- D4's single-pipeline claim: `buildOpenCmd` assigns no environment variable, and adding `--dangerously-skip-permissions` adds none, so `sh.OrElse(resumeLine, fresh)` keeps joining two pipelines.
- D5's "no `--settings` from shuttle": true, `buildOpenCmd` passes none, unlike `buildLaunchCmd`.
- D5's "What it lets through: every tool without a permission dialog" overstates slightly, since Claude Code still raises some dialogs in bypass (`claudeengine/doc.go` records the computed-path `rm` one); an overstatement of exposure leaves the bound conservative, so it is not a defect.
