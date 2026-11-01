```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced every existing code path that touches the state D1 to D8 add or change.

**Findings**

1. **BLOCKING — D6, D7: `{strand}` read from `@strand` gives the role, not the full name.**
   - `setStrandPaneOptionsLocked` (`internal/reedengine/paneoptions.go`) sets `@strand` to `strandLabel(s.Name)`. That is the parsed role segment, and only an unparseable name keeps its full text.
   - D6 says `{strand}` is "the full strand name in the clicking client's active pane". D7 says the verb reads it from the client's active pane, and it may read no reed state. Read that way, it yields `agent` or `plan`, not `&lt;short&gt;:&lt;slug&gt;:&lt;role&gt;`.
   - The record must name another source of the full name, for example a new pane option set beside `@strand`, or reed state looked up by pane id. Either one changes D7's "reads no reed state" or adds a pane option.

2. **MEDIUM — D7, D5: a second documented invariant breaks without the record saying so.**
   - `internal/reedengine/doc.go:78-79` says `@strand` is display only: "no Go decision reads" it. D7's verb reads it, for the `{strand}` fill and its "no `@strand`, run nothing" check.
   - D5 acknowledges the spec option as decision-bearing but not the `@strand` read. The record should state that both invariants change. That covers the `@strand` read or its replacement from finding 1, and the follow/bar doc lines that call `@lym_follow` "the one decision-bearing session option".

3. **MEDIUM — D1, D7: an older binary booting a session undoes line 3, the click branch and line 2.**
   - `pinGeometryOptionsLocked` is called from the boot path (`internal/reedengine/lifecycle.go:587`) and the attach pre-flight (`internal/reedengine/attach.go:98`). It always pins the global `status-format[1]` and calls `pinBindingsLocked` directly, without the stale-build check.
   - When an older lym boots a session, its old `status-format[1]` comes back for every session, so `follow` shows on line 2 and line 3 at once. The `MouseDown1Status` binding is also replaced by one with no `lymbtn*` branch, so button clicks do nothing.
   - The bindings heal at the next reed op of a newer binary, because `repinStaleBindingsLocked` runs in `withOpLock` (`internal/reedengine/lock.go:155`). `status-format[1]` heals only at the next new-binary attach or boot.
   - The record is silent on this mixed-binary window. Today it is reachable because pane binaries are pinned per strand and note `run-pins-its-version` is not landed. The record should state the window and its bound, or put an op-lock repin of the status formats beside the bindings stamp.

4. **LOW — D5: config edits wait for the next attach, though the op lock reaches every op.**
   - Rows render only in the two pin sites. `repinStaleBindingsLocked` already runs at every op, but D5 adds no re-render there, so "next attach or boot" is the only refresh.
   - A beam marked after boot (`fabricengine.MarkBeam`) keeps the task group until then. This is stated as an assumption, so it is consistent, but CANCEL then shows on a session that is now a beam.

5. **LOW — D8: the Windows template is not covered.**
   - `template_windows.yaml` must carry the default set too.
   - Under pwsh, `cd {beam} &amp;&amp; …` needs pwsh 7, and the quoting differs. The record names only "reed's pane shell dialect" and does not mention the Windows template.

**Paths traced**

- D1 `status 3` / `status-format[2]`:
  - Boot pin (`lifecycle.go:587`): survives.
  - Attach pre-flight (`attach.go:98`): survives.
  - Revive: goes through each worktree's boot, survives.
  - `tmux_conf` is sourced before the pins, so reed's values win: survives.
  - Older-binary boot: breaks line 2 (finding 3).
- D1 follow on line 3:
  - Follow click branch and `followClickCommand`: survive, with the range unchanged.
  - `@lym_follow` set on at boot (`lifecycle.go:618`): survives.
- D2 / D3 config load:
  - `MissingKeys` treats a list element as satisfied by its owning key (`internal/yamlengine/reconcile.go:132-177`), so entries without `confirm` or `window` load: survives.
  - The machine layer applies to hub-wide `desktop`: survives.
  - `desktophook.LoadConfig` used by `AfterWindow`: unaffected, since validation happens at render.
- D4: `hubgeom.ReedGeometry` lazy closure: survives. `ParentOf` reads the origin record: survives.
- D5 rows:
  - The status format is expanded in the client's active-pane context, and `@strand` resolves there: survives.
  - The session-window-changed hook only clears `@lym_unseen`: survives.
  - Every op re-pins bindings but not rows: no conflict (finding 4).
- D6 / D7 `{strand}` source: `@strand` holds the role only, so this breaks (findings 1 and 2).
- D7 click branch:
  - The bindings re-pin at each op carries the new branch to older servers: survives.
  - An older binary's boot re-pin: breaks until the next newer op (finding 3).
- D8 templates: POSIX is covered; Windows is not (finding 5).

**Tool calls**

1. Grepped the pin callers, other writers of the status options and the `@strand` setters, to find every path writing the D1 and D5 state. My context held only the definitions.
2. Read `strandLabel`, the op-lock repin site, the boot pin site and the revive hook, to check what `@strand` holds and when the pins and repins run.
3. Grepped `yamlengine` for list handling and other `@lym_follow` readers, to check whether a button entry missing optional keys loads.
4. Read `internal/yamlengine/fill.go`'s shape helpers, to find the list-fill rule.
5. Grepped `fill.go` for the sequence rule, which said a missing key inside an element is left for `MissingKeys`.
6. Grepped `MissingKeys`' element matching, which showed an element path is satisfied by its owning key.
