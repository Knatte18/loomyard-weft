```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set. I reran the searches for F1, F2, F4 and F7. F3, F5, F6 and F8 are single-site changes and need no set search.

**Unchecked:** F3, F5, F6 and F8.

**Findings**

- **LOW**: `internal/shuttleengine/doc.go:27` and `internal/shuttleengine/passages.go:11`. They carry the same stale claim as F7's `claudeengine/doc.go`. Both still say the passages are those "a hub's Bolt checkout holds". `Passages` now also returns the shipped layer's `claude-prompt` entries from the binary. The fix reworded only the `claudeengine/doc.go` sentence.
- **LOW**: `internal/heddlecli/verbs.go:127-133` (`getCmd`). F1's premise says `get` and `list` show two entries under one slug. `list` shows both, but `get` loops `layers.Ordered()` and returns the first match, which is the shipped entry. A colliding repository entry cannot be read with `get`. The fix covered `set`, `remove` and `list` only. `getCmd`'s `Long` text and the new CLI test do not mention it.

**Checked and covered**

- **F1:** `refuseShipped` has two callers, `Set` and `Remove` (`store.go:255`, `store.go:302`), and both were updated. The user-facing refusal text appears in `verbs.go:155` and `:239`, `heddlecli/doc.go:9` and `shipped.go:114`, and all four were updated. `heddleengine/doc.go:3` ("read-only") is still true of the shipped layer.
- **F2:** `ReadLayers` has three callers: `render.go:162`, which feeds `Render`, `Show` and `Passages`, and `verbs.go:94` and `:128`, which are `list` and `get`. `Render` warns and `list` reports `warnings`. `Show`, `Passages` and `get` silently ignore a bad override file. That matches the fix as the review proposed it.
- **F4:** only `pattern/PATTERN-heddle-instructions.md` and the `heddleengine` files name `heddle-lym-override.json` or `OverrideFileName`. No other site needed a change.

**Tool calls**
1. grep for `ReadLayers`, `refuseShipped` and `OverrideWarning` over `.go` files, to list the callers of F1 and F2 (the changed files in my context do not show the other packages).
2. grep for the refusal wording and "Bolt checkout holds" over `.go` and `.md` files, to find twin sites for F1 and F7.
3. grep for `heddle-lym-override`, `OverrideFileName` and `lym_override` outside `internal/heddle*`, to find other references for F4.
