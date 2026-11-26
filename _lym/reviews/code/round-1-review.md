---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/shellguard/rows.go#hubTmuxServerCommandPattern
    summary: "The hub-tmux-server pattern misses clustered getopt flags (tmux -uL lym-x, tmux -2S /tmp/tmux-1000/lym-x) and a quoted -S path whose directory holds a space, and the row comment does not name either as a way past it"
    fix: "In hubTmuxServerCommandPattern, accept a flag word that opens with - and a run of tmux's flag letters before the L or S (for example -[A-Za-z0-9]*L and -[A-Za-z0-9]*S), so a clustered flag matches. Either let a socket path inside a quote hold whitespace, or add the spaced quoted path to the row comment's list of spellings that pass (beside script file, eval, variable, TMUX and symlink). Add deny rows to TestRows_PatternsDenyAndAllow in internal/shellguard/rows_integration_test.go for tmux -uL lym-hub-1a2b3c4d ls and tmux -2S /tmp/tmux-1000/lym-hub-1a2b3c4d ls, and either a deny or an allow row for the spaced quoted path matching the choice."
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shellguard/rows_integration_test.go:687
    summary: Neither new row is tested inside a double-quoted sh -c script, the third scope commandRowPattern renders, and no row covers the single-quote opening hubTmuxServerCommandPattern accepts after -L or -S
    fix: "In TestRows_PatternsDenyAndAllow, add deny rows {\"session-bus\", \"in_a_double_quoted_shell_script\", `bash -c \"busctl --user list\"`, true}, {\"hub-tmux-server\", \"in_a_double_quoted_shell_script\", `sh -c \"tmux -L lym-hub-1a2b3c4d ls\"`, true} and {\"hub-tmux-server\", \"single_quoted_named_server\", `tmux -L 'lym-hub-1a2b3c4d' ls`, true}, beside the existing rows of each guard."
  - id: F3
    severity: LOW
    class: scope
    origin: lens:code-scope
    location: internal/shellguard/rows.go#sessionBusCommandPattern
    summary: The session-bus row refuses dbus-send with an explicit --system, which the board entry does not license; it asks to refuse dbus-send --session, and the diff widens that to every dbus-send
    fix: "Make sessionBusCommandPattern let through a dbus-send whose argument words on the same command include --system (keep refusing dbus-send with --session or with no bus flag), update the dbus-send sentence of the pattern's doc comment and of the summary's design decisions to say so, restore dbus-send on the system bus to the machine-sleep row comment's list of sleep routes that pass, and add an allow row {\"session-bus\", \"dbus_send_system\", \"dbus-send --system --dest=org.x / x.y\", false} to TestRows_PatternsDenyAndAllow."
  - id: F4
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/landing/summary.md
    summary: "The change description claims the RC click's argv and errors are unchanged, but clientAgentPaneVia's not-attached and unreadable-row errors now carry a new 'read the active pane of client' wrapper, and its Merge danger blast radius names only shellguard although the diff also changes reed switch, button clicks and the RC click"
    fix: "In _lym/landing/summary.md, replace \"Its argv and errors are unchanged.\" with a sentence saying its argv is unchanged and its not-attached and unreadable-row errors now read 'reed: read the active pane of client \"<c>\": ...'; and change the Merge danger blast radius to name reed switch, status-bar {strand} buttons and the RC click as well as shellguard."
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: contracts/stencils/sampler/sampler-template-agent.md:43
    summary: The sampler stencil, the one session the hub-tmux-server row newly reaches, lets a trial server name open with lym-, which the row now refuses with a steer pointing at lym's verbs; the row comment's claim that those servers carry other names rests on nothing
    fix: "In contracts/stencils/sampler/sampler-template-agent.md line 43, add that the trial server's name never opens with lym-, the prefix every hub's server carries, keeping the existing requirement that it holds the slug."
---

### [LOW] hub-tmux-server misses clustered flags and spaced quoted socket paths

**Location:** `internal/shellguard/rows.go#hubTmuxServerCommandPattern`
**Issue:** The pattern needs `-L` or `-S` as a word of its own after whitespace, so the valid getopt spellings `tmux -uL lym-…` and `tmux -2S …/lym-…` pass.
`pathChars` excludes whitespace inside a quote too, so `tmux -S "/tmp/my dir/lym-hub-x" ls` passes.
The row is a guardrail, but its comment lists the ways past it and leaves these two out.

### [LOW] New rows untested in the double-quoted script scope and with a single-quoted -L name

**Location:** `internal/shellguard/rows_integration_test.go:687`
**Issue:** `commandRowPattern` renders a separate alternative per scope, each with its own `argument()` reading.
Sibling rows test the double-quoted `sh -c "…"` scope; the two new rows test only the top level and a single-quoted script.
The `'` branch of `(\\"|')?` in `hubTmuxServerCommandPattern` has no row either.

### [LOW] session-bus refuses dbus-send --system beyond the entry

**Location:** `internal/shellguard/rows.go#sessionBusCommandPattern`
**Issue:** The board entry names `dbus-send --session`.
Refusing a bare `dbus-send` follows the entry's intent, since it defaults to the session bus.
Refusing an explicit `--system` call does not, and no added entry or `Pattern-Departure:` trailer licenses it.

### [LOW] Change description misstates the RC error text and the blast radius

**Location:** `_lym/landing/summary.md`
**Issue:** At the base, `clientAgentPaneVia` returned `reed: no client "X" is attached` and `reed: client "X" has an unreadable list-clients row`.
It now wraps the helper's error, giving `reed: read the active pane of client "X": no client "X" is attached`, which `internal/orchcli/remotecontrol.go:195` shows on the operator's status bar.
The summary says the errors are unchanged.
Its `## Merge danger` section names only shellguard as blast radius, while the diff also changes how `reed switch`, `{strand}` buttons and the RC click pick a client's session.
The door call of `two-way` holds: the diff changes no record or board format another binary reads, no config meaning, deletes nothing and pushes nothing.

### [LOW] Sampler stencil does not keep trial servers off the lym- prefix

**Location:** `contracts/stencils/sampler/sampler-template-agent.md:43`
**Issue:** The stencil asks for `tmux -L <name>` with a name that holds the slug and says nothing about its opening.
A name such as `lym-<slug>-trial` is now refused, and the steer then names lym's verbs, a wrong way forward for a trial server.

## Rejected

- `shellguard`'s tier-1 test importing `reedengine` breaks `PATTERN-leaf-packages`: rejected; the package's enforcement, `TestLeafInvariant_AllowlistOnly`, scopes the closed set to non-test files, the package's tests already import testkit packages, and the production imports and `doc.go`'s import sentence still hold.
- `hub-tmux-server` also refuses the session's own hub server, not only another hub's: rejected; a static pattern cannot tell them apart, raw tmux is already refused in every session but the sampler's, and the summary states the call.
- Installing both rows in the orch (`SwitchNone`): rejected; sibling rows follow the same every-run convention, and the entry's purpose holds.
- Another read on the switch or button path still going through `display-message -c` without `-t`: none found; the remaining `display-message -c` calls only show messages, and confirmed and answered clicks carry the strand from the prompt.
- An empty trailing format splitting into too few fields: rejected; `TmuxCmd.output` does not trim, so `c1\t` splits into two fields, and `remotecontrol_test.go` and `TestRunButtonVia` feed that shape.
- Loosened checks in the reedengine or claudeengine tests: none; the raised counts follow the two new rows, and `TestRunButtonVia`'s message check became stricter.
- The tier-2 tests passing on the old read: rejected; in each, the client attached last is the other session's, so the old `display-message -c` read would fail the switch target and the first click's strand.
- Duplicate client name in `clientAgentPaneVia`'s wrapped error (code-correctness NIT): folded into F4, whose fix covers the description; the wording itself is no defect.
