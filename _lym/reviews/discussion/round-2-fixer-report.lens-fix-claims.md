```yaml
lens: fix-claims
outcome: reported
```

Task: check that the code facts in the fix's added or rewritten sentences hold at this worktree's HEAD.

1. MEDIUM. D8, sixth bullet, the added sentence "Before any walk has run, beam's `LoadConfig` applies the same `Migrate` to each layer in memory before `configengine.Load` drops the retired key".
   Summary: the sentence says `LoadConfig` acts on each layer before `Load` drops the key, but `LoadConfig` has no access to the layers at that point. It is the step that closes F1, and nothing in the record names the new hook it needs.
   Evidence: `internal/dobbyengine/config.go:78-79` shows `LoadConfig` calling `configengine.Load(dirs, ConfigSpec())` directly. That function reads the layers itself, merges them into `stack.Doc` and calls `DropRetired(stack.Doc, spec)` (`internal/configengine/config.go:274`), with no hook in between.
   Beam has no `Migrate` today: `internal/configreg/configreg.go:184-216` lists the modules that set `Migrate:`, and beam is not among them.
   The sentence should name the mechanism: a migrate hook in `configengine.Load` or its spec that runs per layer before `DropRetired`, or `LoadConfig` reading and migrating the layers itself.

2. The other changed claims hold:
   - "a call naming no package and lacking `--upkeep` is not registered": `internal/gatecli/testmap.go` contains no `runproc` reference, so a hand call registers nothing today. Only the dobby registers one, at `internal/beamcli/dobbydeps.go:614`.
   - "`test_map_every` is today `CategoryNone`": confirmed at `internal/dobbyengine/config.go:64`.
   - D9's "tries that lock again without waiting": this matches today's non-blocking `TryAcquireWriteLock` at `internal/gatecli/testmap.go:471`.

Tool calls:
- Grepped `gatecli/testmap.go` for runproc, beam's `LoadConfig` and the `configreg` `Migrate` users, because my context had none of their text.
- Read the body of `LoadConfig`, `configengine.Load` around `DropRetired`, and the `configreg` Module docs and its `Migrate:` list, to check how the in-memory migration would order against the drop.
