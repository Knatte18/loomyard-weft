```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked every bound, guarantee and claim about existing code in `decision-record.md`. Five claims break.

**Claims attacked**
- D1 `status` 3 and `status-format[2]` `-g` beside the existing two lines: held (windowsize.go:73-77 pins per index `-g`).
- D1 stale re-pin restores the newest binary's pins after an older binary pins its own: held (bindings.go:55-72, `bindingsStale` is stale when the pinned time is not after the running build's time). The older-binary window and `status 2` are stated in the Bound.
- D2 "the local layer refuses it" (Executable): held (configengine/layers.go `ValidateLocal`, category ≠ None is refused).
- D2 "`LoadConfig` never interprets" an untyped node: held (reedengine/config.go:101-117 and desktophook/config.go `LoadConfig` decode with `yaml.Unmarshal` only).
- D2 "a `buttons` key that is not a mapping … fails that config's load": held (yamlengine/fill.go:113-116).
- D2 "a group written as a mapping fails that config's load": **broken** (F2).
- D3 each group is one list unit, so the machine layer admits a non-empty list against `[]`: held (yamlengine/layers.go `LeafPaths` takes a sequence as one unit; `ValidateMachine` compares mapping-or-not).
- D3 machine file path and edit route: held (configengine/layers.go:102).
- D4 `PaneCwd` already equals `AnchorPath`: held (hubgeom.go:89-90).
- D4 `Resolve` refuses a cwd that is not the anchor: held (lymcwd.go:80-90).
- D4 `ParentOf` names an absent beam, and `NamedWorktree` builds its path without checking it exists: held (parentof.go:13-17; lymcwd.go `NamedWorktree` doc).
- D4 cost "one worktree listing": **broken** (F5).
- D5 `lymbtn&lt;N&gt;` fits the 15-byte range cap: held.
- D7 "exits 0 on every path" keeps view mode shut, and its acceptance criterion "no click … opens a view-mode pane": **broken** (F1).
- D7 tmux 3.6 opens view mode for a background `run-shell` that exits non-zero with no output: held. My probe on tmux 3.6 with `run-shell -b 'exit 3 &gt;/dev/null 2&gt;&amp;1'` left `pane_in_mode=1`.
- D7 the run directory keeps a task worktree from being any button process's cwd, because CANCEL removes the task worktree: **broken** (F3).
- D7 "the verb's log lands in that anchor's logs": held. The logger's sink resolves from cwd through `lymcwd.Resolve` (logger/sink.go:171-182, `LogsDir`:33-39).
- D7 a client gone after CANCEL only fails the message: held (pairteardown/cancel.go `CancelResult.SessionEnded`).
- D7 Bound "a direct call from a strand keeps that strand's identity" and "under its own caller identity": **broken** (F4).
- D7 a carried strand name must pass the grammar, so a direct `--confirmed` call adds nothing over running the command itself: held.
- D8 `lym reed window … --title`: held (reedcli/window.go:104).
- D8 CANCEL from a beam that is being landed is refused, naming the worktree to re-run from: held (battencli/refusal.go:80-88).
- D8 `cd &lt;dir&gt; &amp;&amp; &lt;cmd&gt;` works under bash and pwsh 7: held. pwsh 7 has `&amp;&amp;`.

**Findings**

**F1 — MEDIUM, design, D7 (decision-record.md:115-116, 184).** Exit 0 from the verb does not keep view mode shut.
- tmux reads the exit status of the shell line `run-shell` runs, not the verb's.
- The line starts with the absolute path `os.Executable` gave the pinning binary (reedengine/lock.go:76, panebin.go `composeFollowCommand`).
- Counterexample 1: that path is removed or replaced by a non-executable file after pinning. The shell then exits 127 (or 126). Counterexample 2: the verb panics (exit 2) or is killed by a signal.
- My tmux 3.6 probe confirms a non-zero background `run-shell` opens view mode.
- So the D7 acceptance criterion "no click, refused or failed included, opens a view-mode pane" does not hold.
- Fix: compose the pinned click line, and the confirmed line, so the shell itself ends in exit 0 in reed's shell dialect. That covers every path, the verb's own included.

**F2 — MEDIUM, design, D2 (decision-record.md:49, 179).** The claim "a group written as a mapping … fails that config's load" is false for reed's base and worktree layers and for desktop's hub file.
- Evidence:
  - `fillMapping` checks only template mapping nodes, plus scalar-against-mapping; a sequence template node has no case (yamlengine/fill.go:110-123).
  - `MissingKeys` skips element paths under a present key (yamlengine/reconcile.go:177).
  - `warnUnknownKeys` only logs a Warn (configengine/config.go).
- Counterexample: a worktree `reed.yaml` holding `buttons: {task: {label: x, run: y}}` loads, and the render receives a mapping group.
- Only desktop's machine layer refuses it: `LeafPaths` descends the mapping to `buttons.task.label`, which `ValidateMachine` rejects as "not in the template", so desktop's whole load fails.
- Fix: state the real rule. In reed's layers and desktop's hub file, a group of any shape loads and the render skips it with the Warn. Only the machine layer refuses a mapping group, which fails desktop's load and so `afterWindow`. Correct the D2 acceptance criterion to match.

**F3 — LOW, design, D7 (decision-record.md:113).** The premise "CANCEL removes the task worktree" is false, and the guarantee "no button process holds a task worktree as its cwd" is broken by the record's own default set.
- `lym batten cancel &lt;slug&gt;` keeps the pair by default; only `--remove` removes it (battencli/cancelcmd.go, `cancelCmd` Long: "The pair is kept by default").
- D8's `agent` entry runs `cd {worktree} &amp;&amp; …` (decision-record.md:143), so its shell holds the task worktree as cwd.
- The run directory is still justified by `ResolveBattenPrime` (D8 rationale, line 150).
- Fix: drop the Windows-removal premise and the "no button process" claim, and keep the run directory on the batten-location ground.

**F4 — LOW, design, D7 Bound (decision-record.md:124, 130).** "A direct call from a strand keeps that strand's identity" fails on the confirm path.
- Counterexample: a strand calls `lym reed button` without `--confirmed` on a `confirm` entry and names the operator's client. The verb issues `confirm-before` there.
- On `y`, tmux's server runs the re-run line with its stripped environment (reedengine/lifecycle.go:663-675), so the entry runs as the operator. Tender's identity gate, `refuseCaller` (tendercli/cli.go:57-61), then lets through a cross-pair pause.
- `confirm-before` also takes the client's next keystroke, so an operator typing `y` into a pane confirms it.
- This adds no reach beyond the socket, which can already `run-shell`, but the Bound misstates it.
- Fix: say in the Bound that an unconfirmed direct call can raise a prompt on any client, and that a `y` there runs the entry as the operator.

**F5 — NIT, consistency, D4 (decision-record.md:72).** "One worktree listing" understates the cost.
- `ParentOf` needs the prime's `Location`. A task's geometry gets it only through `fabricengine.PrimeName`, which spawns `git worktree list` (hubgeom.go `isPrimeWorktree` doc).
- That comes on top of `ParentOf`'s own `fabricengine.List` (parentof.go:36), and a missing origin record adds a board read (parentof.go:26-30).
- Fix: state the cost as two listings, or derive the prime's location without the spawn.

**Tool calls**
1. Grepped `internal/reedcli/window.go` for `--title`: D8's flag was not in the manifest.
2. Grepped `internal/logger` for `Arm`: needed where the verb's log lands, not in the manifest.
3. Grepped `internal/logger/sink.go` for the sink-directory resolution: same reason, one hop further.
4. Ran a tmux 3.6 probe on a private socket (`run-shell -b` exiting 3, then read `pane_in_mode`): to check D7's view-mode claim on the real substrate.
5. Grepped `internal/reedengine` for the `executablePath` source and the repo for prebuilt binaries: needed what the pinned binary path is, for F1.
