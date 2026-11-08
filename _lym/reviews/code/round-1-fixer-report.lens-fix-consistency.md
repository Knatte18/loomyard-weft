```yaml
lens: fix-consistency
outcome: reported
```

No findings. Both doc edits agree with `pattern/PATTERN-config-layers.md` lines 44–48, which already say the machine layer applies to hub-wide and per-worktree modules alike.

The remaining "hub-wide module" mentions are about hub-file editing and the worktree layer, so they stay accurate. The codex wording at `docs/overview.md:693` matches the new line 513.

Calls made:
- `grep` for "hub-wide module" and "machine override" across `.md` and `.go` files: to find twins of the two edited lines that my context lacked.
- `Read` of `pattern/PATTERN-config-layers.md` lines 38–57: to check the machine-layer rule against the reworded lines.
