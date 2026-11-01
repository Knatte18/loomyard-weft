```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker lens. I attacked every bound, guard, guarantee and code claim the record states.

**Claims attacked**
1. D7 Bound, "only an operator's click runs it": broken. `lym reed button` is a CLI verb that any agent pane can call directly.
2. Support log Turn 1, "reed's server env scrub strips engine keys only, so … carry a strand name into `run-shell`": broken. `stripAgentNameEnv` already strips both variables.
3. D7 Bound, "the added reach is the caller identity alone": broken. The verb also gets around the shell guard on the strand-name variable and the `confirm` prompt.
4. D7 Bound's list of writers ("tracked `reed.yaml` or … the tmux socket"): broken. Other layer files also decide what a button runs.
5. D7, "the verb's stdout and stderr discarded so `run-shell` opens no view-mode pane": broken by the tmux 3.6 probe.
6. D2, "No reed verb fails on a button": broken by a type-level shape error.
7. D2's `#` bar keeping labels and prompts from injecting format: held for format and style injection, broken for `%` strftime.
8. D4/D6/D8, `{worktree}`, `{beam}` and the spec's worktree as the place lym verbs run from: broken in a subpath-anchored hub.
9. D4 `{beam}` for a task: unstated when `ParentOf` names a beam pair that is absent here or being landed.
10. D7, "a worktree gone by then, as after CANCEL, only fails that message": broken on Windows.
11. D1 Bound on older-binary windows: partly broken, because it names only sessions an older binary booted.
12. D3 shape check admitting a list unit against an empty template list: held (`LeafPaths` treats a sequence as one unit; `ValidateMachine` compares mapping or not).
13. D2 `CategoryExecutable` layer rules: held. Local refuses it; default and worktree layers hold it for the per-worktree reed module.
14. D3 `CategoryMachine` only through the machine layer: held. The machine layer stacks only for a hub-wide module told both Anchor and Board.
15. D5, ranges and styles out of `#{E:}` are drawn: held. Expansion happens before drawing, and the probe shows `#{E:}` expands its value.
16. D5 15-byte `range=user` name and `lymbtn<N>`: held.
17. D5, "one chained invocation so no click sees a row and a spec from different renders": held.
18. D7 hash check refusing a changed spec on the confirmed path: held.
19. D8, "`cd <dir> && <cmd>` runs under bash and pwsh 7 alike": held.
20. Constraints, tmux 3.4 has `run-shell -c`: held.

**Findings**

- **BLOCKING · design · D7 (Bound, environment removal).** The record removes `agentname.StrandNameEnv` and `agentname.ParentEnv` from the command's environment. The premise is false: the tmux server never carries them, because `stripAgentNameEnv` drops both before the server spawns.
  - So the click path already reads as the operator. The removal's only effect is on a direct call: an agent runs `lym reed button --socket … --range lymbtnN --confirmed --hash <hash of the spec run>` and gets that entry's command run as the external caller.
  - That call needs no click and skips the `confirm` prompt. It also gets around the shell guard that refuses an agent command setting or reading `LYM_STRAND_NAME`.
  - Counterexample: an agent pane runs the verb against CANCEL's index and its hash, computed from the spec option it can read over its own socket. The task is cancelled with no operator click and no y/n prompt.
  - The Bound states "only an operator's click runs it" and never names this entry point.
  - Evidence: `internal/reedengine/lifecycle.go:458` and `:660-671` (`stripAgentNameEnv`), `internal/shellguard/rows.go:118-119` (`lymEnvNamesPattern`), `internal/reedcli/follow.go:46` (role audience precedent), `support-log.md` Turn 1 fact.

- **BLOCKING · design · D4/D6/D8 (`{worktree}`, `{beam}`) and D5/D7 (spec worktree).** These values are defined as worktree paths. Every `lym` verb's `lymcwd.Resolve` requires cwd to equal the anchor exactly.
  - In a hub with a recorded subpath anchor, the following all fail with `ErrCwdOutsideAnchor`:
    - running `run` "in the spec's worktree";
    - `cd {beam} && lym batten cancel {slug}`;
    - `lym reed window {worktree}`, which runs `lym reed attach` in that directory.
  - The record itself boots the session "at the anchor" (D7). D8's CANCEL acceptance criterion would fail there.
  - Evidence: `internal/lymcwd/lymcwd.go:58-61` (`AnchorPath`) and `:71-83` (Resolve gate), `internal/lymcwd/anchor.go:161-171` (recorded anchor), `internal/hubgeom/hubgeom.go` `reedGeometry` (`AnchorPath` and `WorktreeRoot` told apart), `lym reed window --help`.

- **MEDIUM · design · D7 (view-mode claim).** Discarding the verb's output does not stop tmux opening view mode.
  - Probe: `run-shell -b -t p: 'exit 3 >/dev/null 2>&1'` on tmux 3.6 left the pane at `pane_in_mode=1 view-mode`.
  - The record states no exit code for the verb's own refusal paths: spec changed, no `@lym_strand_name`, an unreadable spec option, a bad flag. The precedent `reed follow` exits non-zero on error.
  - Each such refusal throws view mode over the operator's active pane, the same pane on the confirmed `run-shell`.
  - Evidence: tmux 3.6 probe above, `internal/reedcli/follow.go` (SetExit on error).

- **MEDIUM · design · D2 ("No reed verb fails on a button").** Validation at render covers only entries that unmarshal.
  - A worktree or hub `reed.yaml` holding `buttons.task` as a mapping or a scalar fails `yaml.Unmarshal` in `reedengine.LoadConfig`.
  - That fails every reed load: `lym reed up`, batten, orch and pair teardown all call it. This is the exact outcome the rejected alternative "Failing `lym reed up` on an invalid button" avoids.
  - `ValidateLocal`'s shape check guards only the local layer, which refuses `buttons` anyway.
  - Evidence: `internal/reedengine/config.go:101-115` (`LoadConfig`), `configengine/layers.go` (shape check only in `ValidateLocal`/`ValidateMachine`), reed `LoadConfig` callers listed by the grep.

- **MEDIUM · design · D4 (`{beam}` for a task).** `ParentOf` deliberately returns the recorded beam slug even when that beam pair is absent on this machine.
  - `lymcwd.NamedWorktree` builds a path with no existence check. `{beam}` is then a nonexistent directory and CANCEL's `cd` fails.
  - A beam being landed passes the `cd`, but `ResolveBattenPrime` refuses it.
  - The record states neither case, and D8's acceptance criterion claims "without the location refusal".
  - Evidence: `internal/hubgeom/parentof.go:16-17`, `internal/lymcwd/lymcwd.go:112-127`, `internal/battencli/refusal.go:80-88`.

- **MEDIUM · design · D7 (CANCEL from a task session on Windows).** The verb runs under `run-shell -c '#{session_path}'`, which is the task's anchor, and runs `run` "in the spec's worktree".
  - Both the verb and its pwsh child hold the task worktree as their process cwd. `cd` in pwsh is Set-Location, which does not move the process cwd.
  - Windows refuses to remove a directory that is a live process's cwd, so the cancel's worktree removal fails.
  - The record treats the worktree as simply "gone by then". D8 ships CANCEL in the Windows template too.
  - Evidence: `PATTERN-pair-teardown.md` (Batten bookend: "never runs from inside that worktree"), `internal/reedengine/lifecycle.go:605-607` (session `-c PaneCwd`), `hubgeom.go` (`PaneCwd` is the anchor).

- **LOW · design · D2 (label and confirm safety).** The `#` bar does not cover `%`.
  - `#{E:}` applies strftime to the option value. Probe: `@x='L%Hx'` gives `#{E:@x}` → `L13x`.
  - A label such as `50%d` renders the date. The `confirm-before -p` prompt takes status-left sequences, so a `%` in a filled value or prompt is rewritten too.
  - This is display corruption only; no range or style injection.
  - Evidence: tmux 3.6 probe above.

- **LOW · consistency · D7 Bound (writers named).** The Bound names only "the tracked `reed.yaml`" and the socket.
  - What a button runs is also set by the hub base `reed.yaml` in the Bolt checkout, `desktop.yaml`'s base, and the untracked `<BoltDir>/.lym/config/desktop.machine.yaml`. Any agent can write that last file.
  - The acceptance argument covers them, but the bound's text does not.
  - Evidence: `configengine/layers.go` `LayerFile` and `MachineFile`, `desktophook/config.go` `ConfigSpec`.

- **LOW · consistency · D1 Bound.** It says "a session an older binary booted keeps `status 2`".
  - An older binary's attach also runs `pinGeometryOptionsLocked`, which sets `status 2` on a session the newest binary booted.
  - The stale re-pin restores only the `-g` pins, never the per-session `status`. That session loses line 3 until the newest binary attaches it again.
  - Evidence: `internal/reedengine/windowsize.go` `pinGeometryOptionsLocked` (session `status 2`, called at boot and attach), `bindings.go` `repinStaleBindingsLocked` (globals only).

Tool calls: none. The inherited context, including the handler's two tmux probes, held everything this lens needed.
