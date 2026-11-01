```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I challenged the proposals behind F1–F5, which were all fixed as proposed.

**Findings**

1. **MEDIUM**: D2, the new sentence "A group of any other shape, a mapping included, loads … with configengine's unknown-keys Warn when it is a mapping".
   - The unknown-keys Warn the F1 proposal describes never fires for a group written as a mapping.
   - Evidence: `warnUnknownKeys` gets its keys from `yamlengine.Prune`.
     `pruneMapping` (`internal/yamlengine/layers.go:396-425`) records a key as removed only when the template lacks it, and it descends only when both sides are mappings (`value.Kind == yaml.MappingNode && templateValue.Kind == yaml.MappingNode`).
     A `task:` written as a mapping meets the template's sequence node, so the key is kept and nothing goes into `removed`.
     No Warn is logged, and only the render's invalid-entry Warn remains.

2. **MEDIUM**: D2. F1's new sentence "In desktop's machine layer a group written as a mapping is refused …, so desktop's load fails" contradicts D2's earlier bullet, which the fix did not touch.
   - That bullet says "skips one of the wrong YAML type with the same Warn …, so such a type error stays inside the button row and leaves whole every load of `reedengine.LoadConfig` and of `desktophook.LoadConfig`, whose `afterWindow` runs on every window lym opens".
   - A group written as a mapping is a group of the wrong YAML type. In the machine layer it now fails the whole desktop load, which includes `window_command` for every window. So the earlier bullet's "every load" is false.
   - The D2 acceptance criterion holds only because it carries the qualifier "the layers' shape check admits". The prose bullet needs the same qualifier.

3. **LOW**: D7, the new run-directory reason "since `lym batten cancel` runs only from the prime or a beam (`ResolveBattenPrime`), so the default CANCEL needs no `cd`" (from F4).
   - The fix's own changes now state this fact three times.
   - Evidence: D8's rationale already says "`lym batten cancel` runs only from the prime or a beam (`ResolveBattenPrime`); a task session's commands run in its beam's anchor (D7), so CANCEL needs no `cd`". The new Auto-mode line from F3 repeats it a third time.

4. **LOW**: D7, the new sentence "The discarded output and the forced zero status together keep view mode shut, whatever the binary or verb does: a removed or replaced pinned binary, a panic and a kill signal included" (from F2).
   - With the forced zero, a click whose pinned binary is gone becomes silent. The verb never runs, so nothing is logged and no status message shows.
   - The record does not state this as a bound. D1's Bound covers dead clicks only in the window where an older binary is still pinned.
   - Evidence: the stale re-pin fires only when the pin is older than the running build (`internal/reedengine/bindings.go:55-72`, as the review cites). A removed binary that is not older leaves the row dead without any notice.

No case found against F3 or F5.

I also checked whether the `;` in `; exit 0` could split a tmux command. It cannot on the click line: the pinned line reaches `run-shell` through `#{E:…}` of an option, so tmux's command parser never sees it (`internal/reedengine/bindings.go:36`, `followClickCommand`).

**Tool calls**
- `grep` of `Prune`/`pruneMapping` and the `CategoryMachine` layer rules: the context did not show how `Prune` treats a mapping set against a sequence template, or how the machine category is admitted.
- `sed` of `pruneMapping` and `grep` of `bindings.go`: the context lacked `pruneMapping`'s body and how the follow line is embedded in the tmux binding.
