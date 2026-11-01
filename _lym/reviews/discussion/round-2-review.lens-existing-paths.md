```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced the existing code paths that write, reset or read each state the record introduces or changes.

**Paths traced, per decision**

- **D1, `status-format[2]`:** an older binary's boot or attach runs `pinGeometryOptionsLocked`, which re-pins `[0]` and `[1]` and calls `pinBindingsLocked`. It never writes `[2]`. The newest binary's next op runs `withOpLock`, then `repinStaleBindingsLocked` (`lock.go:155`), so the result **survives**. The window in between is stated in D1's bound.
- **D1, session `status 3`:** an older binary's `pinGeometryOptionsLocked` writes `status 2` on the session (`windowsize.go:76`). This **breaks** until the next attach by the newest binary, and D1 states it.
- **D1, `MouseDown1Status` button branch:** an older binary's `pinBindingsLocked` rebinds the click without the branch. The newest binary's stale re-pin restores it when `bindingsStale` holds; a binary newer than the stamp is left alone (`bindings.go:55-72`). **Survives**, as stated.
- **D1, `lymbtn*` match against the existing click branches:** user ranges today are only `shell` and `follow`, and window and session ranges read as `window` and `session` (`bar.go:31,34,75,78`; `bindings.go:85-89`). No existing range matches `lymbtn*`, so it **survives**.
- **D2, reed `buttons` key, readers of `reedengine.LoadConfig`:** the callers include tender (`tendercli/deps.go:470`), `batten cancel` (`battencli/cancelcmd.go:405,422`), pairteardown (`pairteardown/teardown.go:561,695,811`), creel delivery (`creelcli/delivery.go:74`), orch routing (`orchroute/live.go:23`), the watchdog (`reedcli/watchdog.go:350`) and SpawnOrder (`hubgeom/spawnorder.go:103`).
  - With `buttons` as an untyped node, a bad entry or group **survives** every reader.
  - A `buttons` key that is not a mapping **breaks** all of them; see finding 1.
- **D2, `lym config reconcile`, `hubreconcile.Ensure` and the sparse strip:** list units are carried whole and a template-equal unit is stripped. **Survives.**
- **D2, local layer:** `ValidateLocal` refuses the key, since its category is not None (`configengine/layers.go:262-305`). **Survives.**
- **D3, desktop `buttons`, readers of `desktophook.LoadConfig`:** `afterWindow` runs on every window lym opens (`desktophook/window.go:44-47`). A wrong entry type inside a list unit **survives** `ValidateMachine`, because a sequence is one unit. A non-mapping `buttons` key fails every window command; D2 states this, and the window is already open, so it is only reported.
- **D3, `lym config desktop --layer machine --edit` and `ValidateMachine`:** the group is one sequence unit with a non-mapping shape on both sides. **Survives.**
- **D4, geometry building by every `hubgeom.ReedGeometry` caller (29 sites):** the closure is lazy and the reads are paid only at boot and attach. The place and origin reads repeat what `parentNameOrEmpty` already does when the geometry is built. **Survives.**
- **D5, session row, spec and `@lym_button_dir` options:** no existing path writes or unsets them. A server rebirth reboots the session and renders again. An older binary's attach leaves them alone. **Survives.**
- **D6, `@lym_strand_name`, set beside `@strand` in `setStrandPaneOptionsLocked`:** that function is also reached through `reassertStrandPaneOptionsLocked` on resume (`paneoptions.go:28-53`), so it is set again with `@strand`. **Survives.**
- **D7, `PersistentPreRunE` of reed:** it resolves the cwd and loads config for every verb not in its skip list (`reedcli/cli.go:142`). D7 says the verb "reads no config", which needs `button` added to that list beside `follow`. That is implementation, and the record's text already implies it. **Survives.**
- **D7, root pre-run:** it runs `seedStencils` and sets up the gates without failing the command (`cmd/lym/main.go:139-170`). **Survives**, but the verb needs `SkipStencilSeedAnnotation` as `follow` has.
- **D7, `lym batten cancel` → `EndForCancel`:** it ends the run's registered processes, sparing the caller and its ancestors, so the unregistered button verb and its shell, as ancestors of cancel, are spared. Then it ends the task's reed session, and the client goes with it. **Survives**, as stated.
- **D7, `lym tender reap` unregistered-orphan rule:** a process qualifies only with a deleted cwd inside the hub and outside every worktree (`tenderengine/processes.go:128`). The button verb's cwd is a live anchor. **Survives.**

**Findings**

1. **LOW · design · `decision-record.md` D2, the shape paragraph ("a `buttons` key that is not a mapping … fails that config's load … Beyond that shape, no reed verb fails on a button").**
   - **Summary:** the record scopes this failure to "that config's load" and reed verbs. A `buttons:` left null, or written as a list, in reed's hub base or a worktree file fails `reedengine.LoadConfig` for every existing reader, not reed verbs alone. That includes `lym tender pause` and `lym batten cancel`, the HALT and CANCEL buttons' own commands, plus pair teardown, creel delivery and orch routing. So a mistake in the key built for halting and cancelling would also disable halt, cancel and mail delivery, hub-wide when it sits in the hub base.
   - **Evidence:** the load fails because `fillMapping` errors when the template holds a mapping and the file does not (`internal/yamlengine/fill.go:87-88,113-115`). The callers are `internal/tendercli/deps.go:470`, `internal/battencli/cancelcmd.go:405`, `internal/creelcli/delivery.go:74`, `internal/pairteardown/teardown.go:561`, `internal/orchroute/live.go:23` and `internal/reedcli/watchdog.go:350`.
   - **Fix direction:** state these readers in D2's bound, or bring the top-level key inside the render-side type check too.

No other path undoes or misreads a decision while the record is silent about it.

**Tool calls**

1. Grep for the writers of `status` and `@strand` and for the status-format pins, to find every existing writer of the options D1 and D6 change.
2. `paneoptions.go`, plus a grep for readers of the status-line count, to find where `@strand` is set and reasserted and whether anything depends on a two-line bar.
3. Callers of `reedengine.LoadConfig` and the orphan rule of tender reap, to find the readers of the new reed key and whether reap could end the unregistered button verb.
4. `statusClickBranches`, the range names in `bar.go` and the call site in `lock.go`, to check how `lymbtn*` dispatch meets the existing ranges and which ops run the stale re-pin.
5. The lock rule of `state.ReadJSON` and the count of `ReedGeometry` callers, to judge the origin and record reads the D4 closure adds at boot.
