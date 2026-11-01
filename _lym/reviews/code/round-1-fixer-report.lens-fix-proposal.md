```yaml
lens: fix-proposal
outcome: reported
```

The plain-cherry-pick claim in F4 holds. Findings below.

**Findings**

1. **LOW**, F1 (`internal/heddlecli/verbs.go` `getCmd` with the new `Set`/`Remove` in `store.go`):
   - Summary: a colliding repository entry is now editable and removable, but `lym heddle get <slug>` can never show its text.
   - Evidence:
     - `getCmd` returns the first match of `layers.Ordered()`.
     - Shipped entries precede repository ones in that list.
     - So `get` prints the shipped entry for the slug, and `list` shows two rows under one slug.
     - Someone cleaning up a collision has no verb that reveals the repository text before editing or removing it.
   - The fixer's own CLI test (`EntryCollidingWithAShippedSlugIsListedAndRemoved`) never calls `get`, so this is unpinned.
   - Mitigation: `list` shows the layer of each row.

2. **LOW**, F2 (`ReadLayers` in `shipped.go`, plus the warnings in `Render` and `list`):
   - Summary: the override-load warning surfaces only in `Render` and `list`.
   - Evidence:
     - `Show`, `Passages` and `get` call `ReadLayers` and drop `OverrideWarning`.
     - So a malformed override silently reverts to the shipped text there.
     - `Passages` feeds a session's appended prompt, so the session gets the shipped text with no signal.
   - This is the degradation the review asked for, so it is only a visibility gap.

**Proposals with no break found:**
- F4 (alternative taken): the cherry-pick claim holds. `replay.go:675` replays a commit that touches an unclaimed path in the `picks` path.
- F5, F6, F7, F8: no break found.
  - The F5 and F6 tests are safe beside the non-parallel test that swaps `shippedStore`, because parallel tests resume only after serial ones finish and the swap is restored by `t.Cleanup`.
  - F8's revert is exact.

**Further tool calls:**
- `sed -n 655,700p internal/fabricengine/replay.go`: to check F4's claim that an unclaimed Bolt path is replayed as a plain cherry-pick, which my context lacked.
