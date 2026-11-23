```yaml
lens: fix-claims
outcome: reported
```

**fix-claims findings**

- **MEDIUM.** The changed text is D4's added sentence: "In the three file layers a key the template lacks otherwise only warns, and the two reconcile commands are the ways forward that warning already names."
  It is false for the local layer. A local file already refuses a unit the template lacks; it does not warn. The warning only covers the default and worktree layers, so it names no way forward for a local file.
  Evidence:
  - `internal/configengine/config.go:300` says: "The machine and local layers are skipped, since each refuses a unit the template lacks."
  - `warnUnknownKeys` sets `wayForward` only for `LayerDefault` and `LayerWorktree` (`config.go:308-315`).
  - `ValidateLocal` refuses on load (`internal/configengine/layers.go:476`).
  The sentence should cover only the worktree and hub base layers. For a local file, D4's check replaces the existing unknown-key refusal, and that refusal now names `lym config reconcile --apply`.

The other added or rewritten claims hold:
- Seed-Child's existing wrapper names `lym board upsert` and `lym config profiles` from the prime worktree (`internal/battenshed/seamchild.go:155`).
- The profile check refuses a write. `checkProfiles` is the profiles module's `CheckLoad` (`internal/configreg/configreg.go:167`), and it runs `PrepareRunSettings` on every block.
- `lym config reconcile --layer default --apply` reaches profiles.yaml. `ReconcileHubWideAt` loops over every `configreg.Modules()` entry (`internal/configsync/configsync.go:433`).
- D7's "as after a skipped hub-wide write" matches the stale stamp and the start-verb retry (`internal/hubreconcile/walk.go:81-83`).
- The prime-seed path goes through `sparseSeed` (`configsync.go:450-492`).
- The `solid.tapestry` block holds `code_max_bounces: 1` (`internal/runprofile/template.yaml:60-62`).

Tool calls:
1. Grepped for the `checkProfiles` call sites, `LayerLocal` handling and the `ReconcileHubWideAt` module loop. My context did not show whether the profile check runs on writes, how the local layer treats unknown keys, or which modules the hub-wide reconcile covers.
2. Grepped for where `checkProfiles` is wired and how `configengine` refuses or warns per layer. The first grep showed no non-declaration call site, so I needed its registration and the local-layer rule.
