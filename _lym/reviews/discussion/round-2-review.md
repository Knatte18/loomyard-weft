---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:49
    summary: D2 says a buttons group written as a mapping fails that config's load, but reed's base and worktree layers and desktop's hub file admit it; only desktop's machine layer refuses it
    fix: In D2, replace the sentence that starts "The layers' shape check still runs before any decode" with this rule. A `buttons` key that is not a mapping (null, list or scalar) fails that config's load, as any key of the wrong shape does. A group of any other shape, a mapping included, loads in reed's hub base and worktree files and in desktop's hub file, with configengine's unknown-keys Warn when it is a mapping, and the render skips it with the invalid-entry Warn. In desktop's machine layer a group written as a mapping is refused as a key the template lacks, so desktop's load fails, adds no machine buttons (D3) and fails that load's `afterWindow` report. In the D2 acceptance criterion, add that a group written as a mapping in reed.yaml loads and is skipped at render.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:115
    summary: Exit 0 from lym reed button does not keep view mode shut; run-shell reads the pinned shell line's status, which is non-zero when the pinned binary is gone or the verb panics or is killed
    fix: In D7, change the click-line bullet so the pinned line itself ends with a forced zero status in reed's pane shell dialect (for example `; exit 0` in POSIX sh, the equivalent in pwsh), on the click line and the confirmed line alike. Keep the verb's own exit-0 rule. Replace the sentence "so discarding the output alone does not keep view mode shut" with one saying both the discarded output and the forced zero status keep it shut, whatever the binary or verb does.
  - id: F3
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:113
    summary: D7 runs task-session commands in the beam's anchor, against the board entry's design that run is run in the session's worktree, and Auto-mode assumptions does not record this departure
    fix: Add a line to `## Auto-mode assumptions` saying that a task session's button commands run in its beam's anchor (the prime's when the beam is absent here), not in the task worktree the board entry names, so the shipped CANCEL works without a cd, and that `{worktree}` names the task for commands that need it (D7). Leave D7's run directory unchanged.
  - id: F4
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:113
    summary: D7's run-directory rationale says CANCEL removes the task worktree and that no button process holds a task worktree as cwd; lym batten cancel keeps the pair without --remove, and D8's agent entry cds into the task worktree
    fix: In D7's run-directory bullet, delete the clause from "so no button process holds a task worktree as its cwd" through "CANCEL removes the task worktree". Replace it with the reason that `lym batten cancel` runs only from the prime or a beam (`ResolveBattenPrime`), so the default CANCEL needs no cd. Keep the sentence on where the verb's log lands.
  - id: F5
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:124
    summary: D7's claim that a direct call from a strand keeps that strand's identity fails on the confirm path, where the operator's y runs the entry as the operator through the tmux server
    fix: In D7, change "and a direct call from a strand keeps that strand's identity" to say this holds only for a direct call that runs the entry itself. Add a sentence to D7's Bound saying that a direct call without `--confirmed`, on an entry with `confirm`, raises the prompt on whichever client it names, and that a `y` there runs the entry as the operator, since the confirmed line runs from the tmux server. This adds nothing over the socket, which can already run commands through `run-shell`.
---

### [MEDIUM] D2 misstates which layers refuse a group written as a mapping

**Location:** `_lym/discussion/decision-record.md:49`, with the D2 acceptance criterion at line 179.

**Issue:** The load's shape check is `yamlengine.fillMapping` (`internal/yamlengine/fill.go:82-124`). It errors only in two cases:
- the template holds a mapping and the file does not;
- the template holds a scalar and the file holds a mapping.

A sequence template node has no case. Each `buttons` group is such a node, so a group written as a mapping passes the check.

Nothing later in the load refuses it either:
- `MissingKeys` skips list-element paths once the list key is present (`internal/yamlengine/reconcile.go:177`).
- `warnUnknownKeys` only logs a Warn.

So `reedengine.LoadConfig` succeeds, and the render receives the mapping group. Only `ValidateMachine` refuses one, because `LeafPaths` descends the mapping to a key the template lacks (`internal/configengine/layers.go`, `ValidateMachine`).

The decision's outcome survives, because the render already skips groups of the wrong type. But the premise is false, and a plan written from it would expect a load failure that never comes.

Three lenses found this independently.

### [MEDIUM] The forced zero status must sit in the pinned line, not only in the verb

**Location:** `_lym/discussion/decision-record.md:115-116`, with the D7 acceptance criterion at line 184.

**Issue:** tmux reads the exit status of the shell line that `run-shell` runs, and that line starts with the absolute path of the binary that pinned the bindings (`internal/reedengine/panebin.go`, `composeFollowCommand`, the model D7 cites). The stale re-pin replaces it only when the pin is older than the running build (`internal/reedengine/bindings.go:55-72`).

That leaves three paths where the line exits non-zero even though the verb's code always exits 0:
- the pinned binary was removed or replaced, so the shell exits 127 or 126;
- the verb panics, which exits 2;
- the verb is killed by a signal.

On tmux 3.6 each of these opens view mode in the operator's pane. The bound-breaker lens confirmed this with a probe: `run-shell -b` exiting 3 left `pane_in_mode=1`.

So the acceptance criterion "no click, refused or failed included, opens a view-mode pane" does not hold under D7 as written.

### [MEDIUM] Running task commands in the beam's anchor departs from the board entry and is not recorded as an auto-pick

**Location:** `_lym/discussion/decision-record.md:113`, and `## Auto-mode assumptions` at line 164.

**Issue:** The board entry's design (operator, 2026-11-01) says ``run` is a shell command, run in the session's worktree``. D7 instead runs a task session's commands in the beam's anchor.

That choice is defensible. The operator's own CANCEL example has no `cd`, and `ResolveBattenPrime` refuses `lym batten cancel` from a task worktree, so the board's two statements cannot both hold.

But the departure changes where every operator-added task button runs, and the run is autonomous. It belongs in `## Auto-mode assumptions` beside the other auto-picks, so the operator sees it there.

### [LOW] D7's Windows rationale rests on a false premise about CANCEL

**Location:** `_lym/discussion/decision-record.md:113`, with D8's `agent` entry at line 143.

**Issue:** `lym batten cancel <slug>` keeps the pair by default; only `--remove` removes it (`internal/battencli/cancelcmd.go`, the `cancelCmd` Long text). D8's CANCEL has no `--remove`.

The claim "no button process holds a task worktree as its cwd" also breaks on the record's own default set: `agent` runs `cd {worktree} && …`.

The run directory is still right, on the ground D8's rationale already gives: `ResolveBattenPrime` (`internal/battencli/refusal.go:57-91`). Only the stated reason is wrong.

Two lenses found this.

### [LOW] D7's identity claim does not hold on the confirm path

**Location:** `_lym/discussion/decision-record.md:124` and the Bound at lines 129-131.

**Issue:** Suppose a strand calls `lym reed button` without `--confirmed` on an entry that has `confirm`. The verb issues `confirm-before` on the client the strand names, and the confirmed line then runs from the tmux server.

The server's environment has `LYM_STRAND_NAME` and `LYM_PARENT` stripped (`internal/reedengine/lifecycle.go`, `stripAgentNameEnv`). So a `y` runs the entry as the operator, and `tender`'s `refuseCaller` (`internal/tendercli/cli.go:57-61`) lets through a pause of another pair.

The reach is no wider than what the socket already gives through `run-shell`. But the Bound's sentence is wrong as written.

## Rejected

- walkthrough 4, a static value holding the literal text `{strand}` gets rewritten at click time: slugs and session names cannot hold braces, and a hub path containing `{strand}` is contrived; the rubric warns against flagging inputs nobody will meet.
- bound-breaker F5, "one worktree listing" understates the cost: how the closure gets the prime's location, and whether that spawns, is Plan-level detail, and D4 accepts the boot-and-attach cost whatever its exact count.
- existing-paths 1, a `buttons` key that is not a mapping fails every reader of `reedengine.LoadConfig`: D2 already says such a key "fails that config's load as any key of the wrong shape does". Every reader of a load shares its failure, and this key is no different from any other reed key of the wrong shape.

## Focus settlements

- Paths and the subpath-anchored hub: settled, it works. The `{worktree}` and `{beam}` values, the spec's run directory and `@lym_button_dir` are all anchor paths, from `AnchorPath` and `lymcwd.NamedWorktree(...).AnchorPath()`. That is what `lymcwd.Resolve` requires of the cwd (`internal/lymcwd/lymcwd.go:81-90`). A task's beam comes from `hubgeom.ParentOf`, which returns the name of the absent beam rather than failing, and D4 covers that case with a presence check. CANCEL from the beam's anchor passes `Resolve` and `ResolveBattenPrime`.
- Trust boundary: F5 corrects the identity claim. Apart from that, the Bound holds:
  - A direct `--confirmed` call runs only an entry the spec holds, with a strand name that passes the grammar check, under the caller's own environment.
  - `stripAgentNameEnv` strips `LYM_STRAND_NAME` and `LYM_PARENT` from the server's environment.
- Exit codes and view mode: F2.
- Config type errors: F1. A wrong type inside a group, and a group of the wrong type, stay inside the button row in reed's layers. A group written as a mapping in desktop's machine layer fails desktop's load.
- `{beam}` for an absent or landing beam, and Windows cwd: an absent beam falls back to the prime's anchor. A beam being landed refuses CANCEL with a way forward, and D8 states this. No CANCEL removes the task worktree, so the Windows hazard D7 cites does not arise; the `agent` entry's shell holds the task worktree only while `shuttle open` runs (F4).
