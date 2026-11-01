```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced the paths that already write, reset or read the state D1 to D8 introduce or change.

**Paths traced, per decision**

- **D1, `status` session option.** Only `pinGeometryOptionsLocked` writes it (`windowsize.go:76`). It runs at boot (`lifecycle.go:587`) and in the attach pre-flight (`attach.go:98`), and it pins `status 2` on whatever session the running binary attaches. **Breaks the bound's wording:** an older binary attaching to a session the newest binary booted also drops line 3. The record names only sessions an older binary booted.
- **D1, global `status-format[0..2]` and the bindings.** The re-pin runs on every op under the op lock (`lock.go:155`) and fires only when the stamp is stale. An older binary's boot or attach calls `pinBindingsLocked` and the status pins unconditionally. **Survives:** the record's bound covers this window.
- **D1, `follow` click branch.** It matches on the range name `follow`, whichever line draws it (`bindings.go` `bindingArgvs`). **Survives.**
- **D2, reed `buttons` key, read by `reedengine.LoadConfig`.** Every caller unmarshals the whole document into the typed `Config` (`config.go:101-115`). Callers include batten's wiring and cancel (`battencli/wire.go`, `cancelcmd.go:405,422`), `pairteardown` (`teardown.go:561,695,811`), `orchcli`, `loomcli`, `tendercli`, `hubgeom/spawnorder.go:103` and configreg's `CheckLoad`. **Breaks:** see finding 1.
- **D2, layer rules.** `ConfigSpec` is per-worktree, so the default and worktree layers hold the key and `ValidateLocal` refuses non-None keys in the local layer (`configengine/layers.go`). **Survives.**
- **D3, desktop `buttons`, read by `desktophook.LoadConfig`.** Its other callers are `afterWindow` (`window.go:45`), which runs after every window lym opens, and configreg's `CheckLoad`. **Breaks:** see finding 1.
- **D3, `ValidateMachine` shape check.** It compares only mapping against non-mapping (`configengine/layers.go`, `ValidateMachine`), so a list unit against the template's empty list passes as the record says. It also passes a scalar such as `task: x`, which then reaches the typed unmarshal.
- **D4, closure.** The geometry comes only from `hubgeom.ReedGeometry` or `standalonegeom` (`standalonegeom/reedgeom.go:50`). A revived session is rebuilt through a hub geometry (`SpawnOrder`), so it carries the closure. **Survives.**
- **D5, row and spec options.** They are written only at the status pins (boot and attach), and revive goes through boot. Agent windows spawned after boot are covered because the agent-or-plain choice is made at draw time from `#{@strand}`. A config edit is picked up at the next attach, as the record states. **Survives.**
- **D6, `@lym_strand_name`.** `@strand` has one writer, `setStrandPaneOptionsLocked` (`paneoptions.go:29`), which is reached from spawn and from `reassertStrandPaneOptionsLocked` at attach (`attach.go`). Pairing the new option with it covers panes an older lym spawned. **Survives.**
- **D7, `run-shell -c '#{session_path}'`.** The session start directory is `PaneCwd`, which is `AnchorPath` (`lifecycle.go:605`, `hubgeom.go:85-89`), as the record says. **Survives.**
- **D7, the verb.** `lym reed button` takes no op lock, so it never triggers the stale re-pin. **Survives:** it needs none.

**Findings**

1. **MEDIUM — design — D2, and D3's desktop key.**
   - Summary: "No reed verb fails on a button" and "a failed desktop load adds no machine buttons" hold only for entries whose YAML types are right.
   - A `buttons` group of the wrong type, such as a scalar or a mapping where a list belongs, fails the typed unmarshal in `reedengine.LoadConfig`. That fails every reed config reader, including batten cancel and pair teardown, so CANCEL itself fails. This is exactly what the rejected alternative "Failing `lym reed up` on an invalid button" was meant to avoid.
   - On the desktop side, the machine shape check admits a scalar against the template list (mapping-or-not only), and the typed unmarshal then fails `afterWindow` for every window lym opens.
   - The record should say how `buttons` is decoded so a type error stays inside the button render, or accept and state that a mistyped file fails these loads.
   - Evidence: `internal/reedengine/config.go:101-115`; `internal/battencli/cancelcmd.go:405,422`; `internal/pairteardown/teardown.go:561,695`; `internal/desktophook/config.go:49-60`; `internal/desktophook/window.go:45-48`; `internal/configengine/layers.go` (`ValidateMachine`: `docIsMapping != (templateValue.Kind == yaml.MappingNode)`).

2. **LOW — consistency — D1 Bound.**
   - Summary: the bound says a session an older binary booted keeps `status 2` until its next attach. In fact `pinGeometryOptionsLocked` re-pins `status 2` on every session any older binary boots or attaches, including one the newest binary booted.
   - Line 3 stays hidden until the newest binary attaches that session; the stale-build re-pin restores only the globals.
   - Evidence: `internal/reedengine/windowsize.go:76`; `internal/reedengine/attach.go:98`; `internal/reedengine/lock.go:155` → `bindings.go` `repinStaleBindingsLocked`.

**Tool calls**

1. grep for writers of `status`/`@strand` and callers of `pinGeometryOptionsLocked`, to find every path writing the D1/D6 state; my context had the definitions but not the callers.
2. grep for `reedengine.Geometry` constructions and a read of the boot and attach pin call sites, to check whether any engine renders without the D4 closure.
3. grep of `hubreconcile` and the worktree config directory, to check whether reconcile materializes `buttons` into a worktree `reed.yaml`. It does not.
4. grep for desktop `LoadConfig` callers and their failure handling, to find which existing paths the D3 key reaches.
