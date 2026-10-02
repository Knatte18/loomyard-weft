# Card 5 — seed-parent

**Edit:**
- `internal/shedrun/seed.go`
- `internal/shedrun/seed_test.go`
- `internal/battencli/arm.go`
- `internal/battencli/arm_seed_test.go`
- `internal/battencli/wire.go`
- `internal/battencli/wire_test.go`
- `internal/loomcli/sharedbootstrap.go`
- `internal/loomcli/sharedbootstrap_test.go`

**Uses:**
- `internal/agentname/agentname.go`

**Intent:**
Record which session spawned a run, so every strand in a task worktree can later be told its parent.

`shedrun.Seed` gains `Parent string` (`json:"parent,omitempty"`), a full agent name.
`WriteSeed` keeps the seed write-once for this field: when an existing seed agrees on recipe, driver and params it is a no-op whatever either side's `Parent` holds, so a later start from a different process never rewrites the field and is never refused over it;
the disagreement refusal keeps comparing recipe, driver and params only.
`ReadSeed` decodes the field; a seed written before this task decodes with an empty parent.
`shedrun` stays the sole encoder and decoder of the struct (Shed Run-Directory Invariant).

Seeding sites read the spawning process's own `os.Getenv(agentname.StrandNameEnv)` — the OS env, never a `.env` overlay, since only reed sets it — and an unset or empty value leaves `Parent` empty:
- `battencli`'s first seeding (`armSeed` in `arm.go`) records it in the batten run's seed;
- the `WriteSeed` closure behind `Seed-Child` in `wire.go` reads the batten run's own seed (`shedrun.ReadSeed(location, slug)`) and copies its `Parent` into the child's seed, so the hub that ran `lyx batten run` is the parent of every process in the task worktree, not batten;
- `loomcli`'s `seedAndCommitBootstrap` passes the caller into `loomSeedFor`, which gains a third argument for it (named `caller`, since its first argument `parent` is the parent branch); on an already-seeded run `WriteSeed`'s no-op keeps the recorded parent.

Tests: `seed_test.go` — the field round-trips, is omitted from the encoded JSON when empty, an old seed without it reads back empty, and a re-write with a different parent but agreeing recipe/driver/params is a no-op that leaves the first parent on disk.
`arm_seed_test.go` — `batten run` under `LYX_STRAND_NAME` (set with `t.Setenv`) records it, and with it unset records none.
`wire_test.go` — the child seed carries the batten seed's parent.
`sharedbootstrap_test.go` — `loomSeedFor` carries the caller, and a first seeding under the env records it while a second start from another env leaves it unchanged.

**ImpactSummary:** Adds an optional, write-once field to seed.json and fills it at the three seeding sites; existing seeds and the disagreement rule are unaffected.

**Commit:** 5: shed seed: record the spawning session as the run's parent
