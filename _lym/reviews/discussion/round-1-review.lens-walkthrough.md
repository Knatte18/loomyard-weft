```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism on one concrete case.

**Walkthrough list**
- D1, an older binary attaches to session S that the newest binary booted, then the newest binary runs its next op. Purpose partly met: the global pins are restored, but S keeps `status 2`, which the Bound does not name.
- D2, a worktree `reed.yaml` holds `buttons: {task: {label: x}}`, a mapping where a list belongs. Purpose not met: `LoadConfig` fails on it, so reed verbs fail.
- D3, `desktop.machine.yaml` holds `buttons.task: [{label: foo, run: echo}]`. Purpose met: `LeafPaths` sees one sequence unit, the template has a sequence there, and the category is machine.
- D4, a task `t` under beam `b` in a hub anchored at a subpath; also `b` absent on this machine. Purpose not met in either case: the value is a worktree root and not the anchor, and the absent case is undefined.
- D5, an agent pane is active and a label holds `%`. Purpose met, apart from the label being strftime-expanded.
- D6, `{worktree}` holds a space, and `{strand}` comes from a pane booted by an older binary. Purpose met.
- D7, CANCEL with `confirm`, clicked in task `t`, then `y`. Purpose not met: a non-zero exit opens view mode, the worktree cannot be removed on Windows, and the variable removal does not do what it claims.
- D8, HALT, CANCEL, term and agent in a task session. HALT, CANCEL and term are not met in a subpath-anchored hub (see D4). The agent entry has a glob NIT.

**Findings**

1. **BLOCKING · design · D4/D5/D6/D8: path placeholders and the command's working directory are worktree roots, but lym's cwd gate needs the anchor.**
   - `{worktree}`, `{beam}` and the spec's worktree are all defined as worktree paths.
   - `lymcwd.Resolve` refuses with `ErrCwdOutsideAnchor` unless cwd is exactly `AnchorPath()` (`internal/lymcwd/lymcwd.go:71-84`, `AnchorPath` at `:59`).
   - Followed through:
     - `cd {beam} && lym batten cancel` refuses.
     - `lym reed window {worktree}` runs `lym reed attach` in that directory (`internal/reedcli/window.go:22`, `:52`), and attach refuses.
     - Every other default command, run "in the spec's worktree", refuses.
   - D7's own click runs at `#{session_path}`, which is the anchor (`PaneCwd: l.AnchorPath()`, `internal/hubgeom/hubgeom.go`; `-c e.geom.PaneCwd`, `lifecycle.go:606`). So the data the click step has and the data the command step uses disagree.
   - Fix: define all three as anchor paths.

2. **MEDIUM · design · D2: "No reed verb fails on a button" fails for a mistyped shape.**
   - Validation runs at render, but `reedengine.LoadConfig` unmarshals the whole document into a typed `Config` first (`internal/reedengine/config.go:101-115`).
   - A `buttons.task` given as a mapping or a scalar fails that unmarshal. That fails every `LoadConfig` caller (`battencli`, `orchcli`, `pairteardown`, `hubgeom/spawnorder.go:103`), not just one button.
   - Fix: decode `buttons` tolerantly (raw node per entry), or state that the failure is accepted.

3. **MEDIUM · design · D4: `{beam}` for a task whose parent beam is absent on this machine is undefined.**
   - `ParentOf` is documented to return the beam slug even when "a member whose beam pair is absent on this machine" (`internal/hubgeom/parentof.go:16-17`).
   - `NamedWorktree` builds the path without checking it exists (`internal/lymcwd/lymcwd.go:112-127`).
   - So CANCEL's `cd {beam}` lands on a directory that does not exist, and the record states no value or bound for that case.

4. **MEDIUM · design · D7: discarding output does not stop view mode.**
   - Probed on tmux 3.6: `run-shell -b 'exit 3 >/dev/null 2>&1'` left the target pane with `pane_in_mode=1 view-mode`.
   - The verb's exit code is unstated on its refusal paths: a changed hash, a missing `@lym_strand_name`, a spec read failure.
   - The existing model verb exits non-zero on error (`internal/reedcli/follow.go`, `SetExit(output.Err…)`).
   - So these clicks open view mode over the operator's active pane, the agent's pane included.
   - Fix: the verb exits 0 after reporting through `display-message`, on both the click line and the confirmed line.

5. **MEDIUM · design · D7: removing the strand-name and parent variables rests on a false premise.**
   - The record assumes the `run-shell` child inherits a strand name from the server environment. It does not: the server spawn already strips `LYM_STRAND_NAME` and `LYM_PARENT` (`internal/reedengine/lifecycle.go:458`, `stripAgentNameEnv` at `:663`).
   - The support log's Turn 1 fact ("strips engine keys only") is therefore wrong.
   - The removal only has an effect when the verb is started by something other than tmux. An agent that runs `lym reed button --confirmed …` itself has its identity stripped.
   - That contradicts the Bound's "only an operator's click runs it", and it gets around the shellguard deny on touching the strand variable (`internal/shellguard/rows.go:119`).

6. **MEDIUM · design · D7/D8: CANCEL from a task session cannot remove the task worktree on Windows.**
   - `run-shell -c '#{session_path}'` starts the verb with its cwd in the task anchor, and the verb runs the shell "in the spec's worktree", also the task.
   - Only the `cd` child moves to the beam, and pwsh's `cd` does not change the process cwd.
   - Teardown ends the pair's reed session, but not this server-owned `run-shell` job (`PATTERN-pair-teardown.md`). Windows refuses to delete a directory that a live process holds as its cwd.
   - D8 ships the set in the Windows template, and its acceptance criterion assumes the removal succeeds.

7. **LOW · consistency · D1 Bound.**
   - An older binary's attach runs `pinGeometryOptionsLocked`, which sets `status 2` on whichever session it targets (`internal/reedengine/windowsize.go`, the `"status", "2"` pin), including one the newest binary booted.
   - `repinStaleBindingsLocked` restores only global state (`bindings.go:166-180`).
   - The Bound names only sessions an older binary booted, and "until its next attach" does not say whose attach.

8. **LOW · design · D4/D5: the lazy-`{beam}` ordering does not fit the closure's stated contract.**
   - The closure returns the placeholder values and the machine's buttons together.
   - Resolving `{beam}` only when an entry uses it needs the entries before the values, and some of those entries are the machine's, which come out of the same closure.
   - The record does not say how the closure learns whether `{beam}` is used.

9. **LOW · design · D2/D5: labels are strftime-expanded on the status line.**
   - Probed on tmux 3.6: an option holding `L%Hx`, read through `#{E:@x}`, printed `L13x`.
   - Validation bars `#` and control characters but not `%`, so a label like `50%` or `%H` renders wrong.
   - The `confirm-before -p` prompt takes the same status-left sequences.

10. **NIT · design · D8.** `--model opus[high]` is an unquoted bash glob, so a file named `opush`, `opusi` or `opusg` in the cwd would replace it.

Tool calls:
1. Grepped `internal/reedcli/window.go` to confirm that `reed window` runs `lym reed attach` in the told directory. My context had only the verb's help text, not where it runs attach.
