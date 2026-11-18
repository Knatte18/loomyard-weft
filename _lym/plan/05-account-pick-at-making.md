# Card 5 — Pick a worktree's account when it is made on the machine

**Create:**
- `plan:internal/shuttleengine#AccountKeeper` -> `type AccountKeeper interface`
- `plan:internal/shuttleengine/claudeengine#Claude.PickAccount` -> `func (c *Claude) PickAccount(keepEntry bool) error`
- `plan:internal/shuttleengines#AccountKeeper` -> `func AccountKeeper(dirs configengine.Dirs, worktree shuttleengine.Worktree) (shuttleengine.AccountKeeper, bool)`
- `plan:internal/shuttleengine/claudeengine#TestClaude_PickAccount` -> `func TestClaude_PickAccount(t *testing.T)`
- `plan:internal/battencli#TestWire_CreateWorktreePicksTheNewWorktreesAccount` -> `func TestWire_CreateWorktreePicksTheNewWorktreesAccount(t *testing.T)`

**Extends:**
- `internal/battencli#battenCLI`
- `internal/adoptcli#adoptDeps`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine/claudeengine/doc.go`
- `internal/shuttleengines/doc.go`
- `internal/adoptcli/doc.go`

**Edit:**
- `internal/battencli#battenCLI.wire`
- `internal/adoptcli#hubDepsAs`
- `internal/adoptcli#runForeground`
- `internal/adoptcli#takeOverRuns`
- `internal/adoptcli#TestAdopt`
- `internal/adoptcli#world.deps`

**Uses:**
- `plan:internal/shuttleengine/claudeengine#pickAccount`
- `plan:internal/shuttleengine/claudeengine#updateAccountFile`
- `plan:internal/shuttleengine/claudeengine#readUsage`
- `plan:internal/shuttleengine/claudeengine#loggedIn`
- `plan:internal/hubgeom#EngineWorktree`

**Intent:**
D3's moments: the task run's Worktree-Create step, a new beam's creation (its pawl run makes the pair through the same Worktree-Create step), and each pair `lym adopt` adds.

`shuttleengine.AccountKeeper` is an optional engine capability; this card gives it `PickAccount(keepEntry bool) error`, and card 9 adds the verbs' methods.
`shuttleengines.AccountKeeper(dirs, worktree)` builds each registered engine through its constructor with a zero config and returns the first that implements the capability; a CLI verb and the wiring reach the claude engine only through it (`PATTERN-shuttle-provider-seam`).

`Claude.PickAccount` loads claude.yaml and returns its error; it is a no-op when fewer than two accounts have positive weight, when the told worktree is the prime or has no name, under a hub pin, or under a beams pin for a beam worktree.
Otherwise it notes the time, then in one `updateAccountFile` hold: calls `Worktree.Names` and drops every entry whose name it lacks, so a worktree made while the pick waited for the lock keeps its entry (D2); keeps the entry and changes nothing more when `Picked` for this name is later than the noted time, `Picked` holding the engine clock's (`now`) time of an earlier pick's write, read under the lock, so two picks at once leave one entry and one increment; keeps the entry and changes nothing more when `keepEntry` is set and the file holds an entry for the name;
otherwise runs `pickAccount` with the fallback on and writes the name's entry and `Picked` time, replacing any earlier entry.
With no candidate it removes the name's entry and logs it at Info, and the worktree runs as today.
A failed `Names` call refuses the pick and writes nothing.

Wiring: `battenCLI` gains a `pickAccount func(*lymcwd.Location, string, bool)` field, which `battenCLI.wire` fills beside its other seams.
Its closure builds the new pair's location, takes `hubgeom.EngineWorktree` of it and calls `shuttleengines.AccountKeeper(...).PickAccount(keepEntry)`.
`wire`'s `CreateWorktree` closure calls it with the slug and `keepEntry` false after `top.Add` succeeds.
Since `wire` fills the field, every existing test that calls `wire` and then drives `CreateWorktree` past the add, or into its `complete` branch, reaches the real pick.
That pick writes nothing on a hub with fewer than two accounts and never fails the create.
Its `complete` branch, which a row re-entered after a crash between `top.Add` and the pick takes, calls it with the slug and `keepEntry` true before `reconcileTaskPair`, so a worktree the crash left without an entry is picked and one with an entry keeps it (D3).
`adoptDeps` gains `PickAccount func(slug string)`, set in `hubDepsAs` the same way with `keepEntry` false; `runForeground` and `takeOverRuns` call it after each `AddPair` that succeeds, so `lym adopt root` and a pair already local pick nothing (D9).
The adopt tests' shared builder `world.deps` sets a `PickAccount` that records each slug, which the new `TestAdopt` rows assert, so no existing adopt test calls a nil func.
Every pick failure is logged at Warn and never fails the create or the adopt: the pick writes only the machine-local account file, and the launch reports a bad config itself (D1).

`TestClaude_PickAccount` is tier0 over a temporary hub with a machine-layer claude file and fake config directories: one positive-weight account writes nothing; the prime, a hub pin and a beams-pinned beam write nothing; a task worktree gets an entry and its account's count rises by one; an entry of a name `Names` lacks is dropped while one `Names` returns that the file did not hold before keeps it; a pick over an existing entry replaces it; a pick whose clock gives a noted time before the `Picked` time a concurrent pick already wrote for the name keeps that entry and adds no increment, which is the state two picks at once leave the second; a failed `Names` call writes nothing; no logged-in candidate removes the entry; with `keepEntry` set a name with an entry keeps it with no increment, and a name without one is picked.
`TestWire_CreateWorktreePicksTheNewWorktreesAccount` is tier1 on a `hubforge` hub whose fabric config lets `top.Add` succeed: the closure calls `pickAccount` once with the slug and `keepEntry` false after the add; a `pickAccount` that logs a failure leaves the create successful; an add that fails calls no pick; a row re-entered over a complete task worktree calls `pickAccount` once with the slug and `keepEntry` true.
It asserts `wire` leaves `pickAccount` non-nil, then overrides the field after `wire`, as the owner-seam tests override `owner`.
`TestAdopt` gains rows: `PickAccount` is called for each slug `AddPair` added, at both sites, and never for a local pair or `root`.

**ImpactSummary:** A new optional capability and two wiring seams; Worktree-Create and adopt gain one logged, non-failing call after each pair is added, and Worktree-Create's already-complete branch one that keeps an existing entry.

**Commit:** 5: pick a worktree's Claude account at Worktree-Create and adopt
