```yaml
lens: fix-proposal
outcome: reported
```

fix-proposal check: I challenged each reviewer proposal the fixer carried out, looking for a concrete case where the change is wrong.

**Proposals challenged:** F1, F2, F3, F4, F5, F7, F8, F9, F10, F11, F12. F6 was carried out with the fixer's own change.

1. **MEDIUM — F4. The guarantee is broader than the code allows.**
   - **Changed text:** D2's new bullet says reed's and desktop's `Config` types decode `buttons` as an untyped node, "so a type error stays inside the button row", and that "This keeps whole every load of `reedengine.LoadConfig` and of `desktophook.LoadConfig`".
   - **Case:** `desktop.machine.yaml` holds `buttons: 5`, so the whole key is a scalar, not one group. `readLayers` runs `ValidateMachine` on the machine layer (`internal/configengine/layers.go`, inside `readLayers` near line 480). Its shape check at `layers.go:345-347` compares the doc's `buttons`, a non-mapping, with the template's `buttons` mapping. It returns MarkInvalid "the desktop template holds a different shape". `desktophook.LoadConfig` then fails before any untyped decode, and so does `afterWindow` on every window.
   - The proposal's guarantee holds only for a wrong-type group or entry, not for a wrong-type `buttons` key.
   - **Fix:** narrow the claim to groups and entries. Alternatively, state that a non-mapping `buttons` in the machine layer is refused by the layer shape check, as any machine key of the wrong shape is.

2. **MEDIUM — F1. Making `{worktree}` an anchor changes `term`'s window title.**
   - **Changed text:** D4 now defines `{worktree}` as the session worktree's anchor path. D8's `term` stays `lym reed window {worktree}`.
   - **Case:** `lym reed window --help` says "The title defaults to the directory's base name." In a subpath-anchored hub, every `term` window is titled with the anchor subpath's base name, the same for every worktree, instead of the worktree's name. Before the fix, `{worktree}` was the worktree root, and the title named the task.
   - **Fix:** ship `term` as `lym reed window {worktree} --title {slug}`.

Tool calls beyond my context:
- I grepped reedengine for `set-environment` and `StrandNameEnv`, ran `lym reed window --help`, and grepped `LoadOrTemplate` for shape checks. This checked whether reed injects the strand name server-wide (F2's premise), what `reed window` does with its directory argument (F1), and whether any layer shape check precedes the untyped decode (F4).
- I listed configengine's functions and its "different shape" sites to find which validators refuse a wrong shape (F4).
- I grepped `readLayers` for its `Validate*` calls to confirm that the machine-layer check runs on load (F4).
