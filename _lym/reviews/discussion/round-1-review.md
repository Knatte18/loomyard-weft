---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md D4 (Values), D5 (button spec), D7 (run directory), D8 (CANCEL, term) and its acceptance criterion
    summary: The path placeholders and the command's run directory are worktree roots, but every lym verb needs its cwd to be the worktree's anchor exactly, so in a subpath-anchored hub every default-set lym command is refused with ErrCwdOutsideAnchor
    fix: In D4, define {worktree} as the session worktree's anchor path (the geometry's AnchorPath, which PaneCwd already equals) and {beam} as the beam's anchor path (the prime's, the beam's own, or for a task the anchor of the worktree ParentOf names, read through lymcwd.NamedWorktree's AnchorPath). In D5, make the spec hold the session's anchor path, not its worktree path. In D7, say the verb runs run in that anchor. Add the evidence to D4's rationale - lymcwd.Resolve requires cwd to equal AnchorPath, and anchors may be a subpath (recorded .lym-anchor) - and change the D8 criterion to name the beam's anchor.
  - id: F2
    severity: BLOCKING
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md D7 (environment removal and Bound); _lym/discussion/support-log.md Turn 1 facts
    summary: Removing the strand-name and parent variables rests on a false premise, since the tmux server already strips both, so its only effect is that an agent calling lym reed button directly, with --confirmed and the readable hash, runs any entry as the operator with no click and no confirm prompt, and gets around the shell guard on LYM_STRAND_NAME, an entry point the Bound denies exists
    fix: In D7, drop the bullet clause that removes agentname.StrandNameEnv and agentname.ParentEnv. State instead that the click path already reads as the operator because reed strips both variables from the tmux server's environment at spawn (stripAgentNameEnv), so a direct call from a strand keeps that strand's identity. Rewrite the Bound to name the direct call - any process that can run lym reed button against the socket can run a spec entry, --confirmed skipping its prompt, under its own caller identity, which adds nothing over running the command itself - and delete the sentence only an operator's click runs it. Correct the support log's Turn 1 fact about the env scrub to say the server spawn strips LYM_STRAND_NAME and LYM_PARENT.
  - id: F3
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md D7 (pinned line and confirm path)
    summary: Discarding the verb's output does not stop run-shell opening view mode, because tmux 3.6 opens view mode in the target pane for a background run-shell command that exits non-zero even with no output, and the verb's exit code on its refusal paths is unstated
    fix: Add to D7 that lym reed button exits 0 on every path once it has reported through display-message - spec changed, no @lym_strand_name, unreadable spec option, a failed command - on both the click line and the confirmed line. Cite the probe as evidence - run-shell -b with 'exit 3 >/dev/null 2>&1' leaves the pane in view-mode on tmux 3.6.
  - id: F4
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md D2 (validation bullet) and D3 (failed desktop load)
    summary: The guarantee that no reed verb fails on a button holds only for entries that unmarshal, because a buttons group of the wrong YAML type fails the typed unmarshal in reedengine.LoadConfig, and so every reed config reader including batten cancel and pair teardown, while on the desktop side ValidateMachine's mapping-or-not shape check admits a scalar group that then fails desktophook.LoadConfig for every window lym opens
    fix: In D2, state that reed's and desktop's Config types decode buttons as an untyped YAML node that LoadConfig never interprets, and that the render decodes each group and entry and skips one of the wrong type with the same Warn as any other invalid entry, so a type error stays inside the button row. Name both readers - reedengine.LoadConfig and desktophook.LoadConfig, whose afterWindow runs on every window - as the loads this keeps whole.
  - id: F5
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md D4 ({beam} for a task) and the D8 acceptance criterion
    summary: The record never says what {beam} is when ParentOf names a beam pair absent on this machine, which ParentOf explicitly allows and NamedWorktree does not check, or when the beam is being landed and ResolveBattenPrime refuses it, so CANCEL's cd fails or meets the location refusal the D8 criterion says it avoids
    fix: In D4, add that when the worktree ParentOf names is not present on this machine, every entry using {beam} is left out of the session's row with one Warn naming the entry and the missing beam. Add to D8's rationale that CANCEL on a task whose beam is being landed is refused by ResolveBattenPrime, which names the worktree to re-run from, and that this refusal reaches the operator as the failure message of D7. Change the D8 criterion to cover a present, non-landing parent beam.
  - id: F6
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md D7 (run-shell -c and run directory) with D8 CANCEL on the Windows template
    summary: On Windows, CANCEL clicked in a task session leaves the lym reed button process, started with its cwd in the task's anchor, and its shell holding the task worktree as process cwd while lym batten cancel removes that worktree, and Windows refuses to remove a directory that is a live process's cwd
    fix: In D7, start the click's run-shell (-c) and run the command from the session's beam anchor ({beam}) in a task session, and from the session's own anchor elsewhere, so no button process holds a task worktree as its cwd. Say that the verb's log therefore lands in that anchor's logs. Then drop the cd {beam} && prefix from CANCEL in D8 and note that {worktree} still names the task for term.
  - id: F7
    severity: LOW
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md D2 (label and confirm validity) and D6 (confirm filling)
    summary: "The # bar does not cover %, and #{E:} applies strftime to the option value (tmux 3.6 probe - an option holding L%Hx expands to L13x), so a % in a label is drawn as a time field, and the confirm-before prompt takes status-left sequences, so the same holds there"
    fix: "In D2, add % to the characters label and confirm may not hold. In D6, double each % as well as each # when a value is filled into confirm. Cite the probe in D2's rationale."
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D3 (last bullet) and D4 (closure failure bullet)
    summary: D4's closure returns the machine's buttons, so a failed desktop load happens inside it, and D4 says a closure failure shows follow alone while D3 says a failed desktop load only drops the machine's buttons
    fix: In D4, say that a failed desktop load inside the closure is logged at Warn and returns no machine buttons, leaving the place and values intact, and that only a failure to read the place or a value shows follow alone.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D4 (first and fourth bullets)
    summary: "{beam} is resolved only when an entry uses it, but the closure returns the placeholder values together with the machine's buttons, and whether {beam} is used depends on those buttons, so the record does not say how the laziness works"
    fix: In D4, make the closure return the place, the three cheap values, the machine's buttons and a separate function that resolves {beam}, which the render calls only when an entry of the composed row uses {beam}.
  - id: F10
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md D7 (Bound, third sentence)
    summary: The Bound names only the tracked reed.yaml and the tmux socket as ways to change what a button runs, but the hub's base reed.yaml in the Bolt checkout, desktop.yaml's base and the untracked desktop.machine.yaml under the Bolt checkout's .lym also set button commands
    fix: In D7's Bound, replace the tracked reed.yaml with every file that defines buttons - reed.yaml's hub base and worktree layers, and desktop.yaml's hub base and machine layer.
  - id: F11
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md D1 (Bound)
    summary: The Bound says a session an older binary booted keeps status 2, but an older binary's attach also sets status 2 on a session the newest binary booted, and the stale-build re-pin restores only the global pins, so that session also loses line 3 until the newest binary attaches it
    fix: In D1's Bound, change the last clause to say that any session an older binary boots or attaches keeps status 2, without line 3, until the newest binary next attaches it.
  - id: F12
    severity: NIT
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D8 (agent entry)
    summary: The agent entry's opus[high] is an unquoted bash glob, so a file named opush, opusi or opusg in the run directory would replace the model spec
    fix: In D8, write the agent entry's run as lym shuttle open --role agent --model 'opus[high]'.
---

### [BLOCKING] Path placeholders are worktree roots; lym verbs need the anchor

**Location:** `_lym/discussion/decision-record.md` D4, D5, D7, D8

**Issue:** `lymcwd.Resolve` (`internal/lymcwd/lymcwd.go:71-83`) refuses with `ErrCwdOutsideAnchor` unless cwd equals `AnchorPath()`, which is the worktree path joined with the recorded anchor (`internal/lymcwd/anchor.go:161-171`).
D4, D5 and D7 use worktree paths, and `reedGeometry` keeps `WorktreeRoot` and `AnchorPath` apart (`internal/hubgeom/hubgeom.go`).
In a subpath-anchored hub, the following are all refused:

- `cd {beam} && lym batten cancel {slug}`;
- `lym reed window {worktree}`, which runs `lym reed attach` in that directory;
- every command run "in the spec's worktree".

D7 itself already starts the click "at the anchor".

### [BLOCKING] The env removal's only effect is an unbounded direct entry point

**Location:** `_lym/discussion/decision-record.md` D7; `_lym/discussion/support-log.md` Turn 1

**Issue:** The server spawn already strips `LYM_STRAND_NAME` and `LYM_PARENT` (`internal/reedengine/lifecycle.go:458`, `stripAgentNameEnv` at `:660-671`), so the support log's fact that the scrub "strips engine keys only" is false.
The removal therefore changes nothing for a click.
It does strip the identity of an agent that runs `lym reed button --confirmed --hash …` itself.
The agent can read the hash's input from the spec option over its own socket.
That call cancels or halts with no click and no y/n prompt, under the operator's identity.
It also gets around the shell guard's refusal of commands that touch the strand-name variable (`internal/shellguard/rows.go:119`).
The Bound says "only an operator's click runs it", which leaves this edge unbounded.

### [MEDIUM] A non-zero exit opens view mode despite the discarded output

**Location:** `_lym/discussion/decision-record.md` D7

**Issue:** Probe on tmux 3.6: `run-shell -b -t p: 'exit 3 >/dev/null 2>&1'` left the pane at `pane_in_mode=1 view-mode`.
The model verb `reed follow` exits non-zero on error (`internal/reedcli/follow.go`).
The record states no exit code for the button verb's refusals, so each refusal can cover the operator's active pane with view mode.

### [MEDIUM] A mistyped `buttons` group fails every reed and desktop config load

**Location:** `_lym/discussion/decision-record.md` D2, D3

**Issue:** `reedengine.LoadConfig` unmarshals the whole document into a typed `Config` (`internal/reedengine/config.go:101-115`).
Its callers include batten cancel, pair teardown, orch and spawn order.
A group of the wrong type fails all of them, which is the outcome the rejected alternative "Failing `lym reed up` on an invalid button" was meant to avoid.
`ValidateMachine` compares mapping against non-mapping only (`internal/configengine/layers.go`), so a scalar group in `desktop.machine.yaml` passes the check.
It then fails `desktophook.LoadConfig`, whose `afterWindow` runs for every window lym opens.

### [MEDIUM] `{beam}` for an absent or landing parent beam is unstated

**Location:** `_lym/discussion/decision-record.md` D4, D8 acceptance criteria

**Issue:** `ParentOf` returns the beam slug even when that pair is absent on this machine (`internal/hubgeom/parentof.go:16-17`).
`NamedWorktree` builds the path without checking that it exists (`internal/lymcwd/lymcwd.go:112-127`).
`ResolveBattenPrime` refuses a beam that is being landed (`internal/battencli/refusal.go:80-88`).
The record leaves both cases for the implementer to discover.

### [MEDIUM] CANCEL from a task session blocks its own worktree removal on Windows

**Location:** `_lym/discussion/decision-record.md` D7, D8

**Issue:** The session's start directory is the task anchor (`-c e.geom.PaneCwd`, `internal/reedengine/lifecycle.go:605-607`).
So the verb process, and the shell that D7 runs in the task worktree, both hold the task worktree as their cwd while `lym batten cancel` removes it.
Teardown ends the pair's session but not a server-owned `run-shell` job.
Windows refuses to delete a directory that a live process uses as its cwd, and D8 ships CANCEL in the Windows template.
The record only says the worktree will be "gone by then".

### [LOW] `%` is strftime-expanded in labels and the confirm prompt

**Location:** `_lym/discussion/decision-record.md` D2, D6

**Issue:** Probe on tmux 3.6: `#{E:@x}` with `@x` set to `L%Hx` printed `L13x`.
A label holding `%` is drawn with a time field substituted.
The confirm prompt takes status-left sequences too.

### [LOW] D3 and D4 disagree on a failed desktop load

**Location:** `_lym/discussion/decision-record.md` D3, D4

**Issue:** The desktop load runs inside D4's closure.
D4 shows `follow` alone on any closure failure, while D3 drops only the machine buttons.

### [LOW] Lazy `{beam}` does not fit the closure's contract

**Location:** `_lym/discussion/decision-record.md` D4

**Issue:** Whether `{beam}` is used depends on the machine entries, and those come out of the same closure that is said to return `{beam}`.

### [LOW] The Bound names too few files that decide what a button runs

**Location:** `_lym/discussion/decision-record.md` D7 Bound

**Issue:** `configengine.LayerFile` and `MachineFile`, read with each module's `ConfigSpec`, show more files that hold button commands than the Bound names.
They are the hub base and worktree `reed.yaml`, and the hub base and machine `desktop.yaml`.

### [LOW] D1 Bound covers only sessions an older binary booted

**Location:** `_lym/discussion/decision-record.md` D1

**Issue:** `pinGeometryOptionsLocked` sets `status 2` at boot and in the attach pre-flight (`internal/reedengine/windowsize.go`, `attach.go`).
`repinStaleBindingsLocked` restores only the global pins (`internal/reedengine/bindings.go`).
So an older binary's attach also hides line 3 on a session the newest binary booted.

### [NIT] `opus[high]` is an unquoted glob

**Location:** `_lym/discussion/decision-record.md` D8

**Issue:** Under bash, a matching file in the run directory would replace the model spec.

## Rejected

- Walkthrough's MEDIUM rating of the env-removal finding: merged into F2 at BLOCKING. The edge lets any agent cancel a task with no click and no prompt, and the Bound states the opposite.
- Bound-breaker's claim that D7's hash check, the D3 shape check and the D5 range claims break: none was claimed broken; the lenses held them, and so does this review.

Note on independence: every fork inherited the handler's context, including the tmux probes and the reading notes, so the agreement between lenses on F1 to F6 is not independent confirmation; each finding was judged here on its cited evidence.
