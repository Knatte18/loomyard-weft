# Decision record — status-strand-by-driver

## Goal

A loom run's tmux session shows exactly one driving surface, chosen by the run's recorded driver.
An `llm`-driven run shows the ly-drive strand and never the `loom-status` strand;
a `go`-driven run shows the `loom-status` strand.
`lyx loom status` stays available as a command for both.

## Scope

In:

- `lyx loom start` ensures the status strand only when the run's driver is `go`.
- `lyx loom start` on an `llm`-driven run removes a status strand the session still holds (from an older build, or from a start that ran before this change).
- `lyx loom step` and `lyx shed step` (loom recipe) stop doing any status-strand work.
  They keep bringing the reed substrate up.
- `CONSTRAINTS.md`'s Driver Choice Single-Site Invariant is reworded so the driver branch selects the run's driving surface (which driver spawns, and whether the session carries the status strand), not only the driver spawn.
- `docs/overview.md`'s `start` description, and any `step` description that mentions the status strand, updated in the same commit.
- Tests for all of the above.

Out:

- Pane height.
  With the status strand gone, the llm stack is ly-drive over the agent, so `collapsed_rows` is the ly-drive height;
  the hub config already sets it (16, weft `main-weft`).
  No new config key and no code change.
- Removing a stray ly-drive strand from a `go`-driven run's session.
- The `lyx loom status` command itself.
- Batten and every other recipe: only loom has a status strand.

## Decisions

- **Driver source for the gate is the value `seedAndCommitBootstrap` already returns in `start`.**
  Rationale: `start` is loom's bootstrap verb and already reads the driver exactly once for its spawn branch;
  reusing that value adds no second read and keeps the Driver Choice Single-Site Invariant's "one read per recipe" intact.
- **`start` splits today's `ensureStatusStrand` call into: reed `Up` unconditionally, then a driver branch.**
  `go` → today's keep/replace/add status-strand logic, unchanged.
  `llm` → remove any strand named `loom-status`.
  Rationale: the llm arm still needs the reed session up for its driver strand, so `Up` must not move inside the `go` branch.
  The branch sits where `ensureStatusStrand` sits today (under the bootstrap lock, before the watchdog spawn and the driver spawn), so the llm driver strand is added into a session that no longer holds the status strand.
- **The llm-arm removal matches by the pinned `statusStrandDisplayName` and removes every strand carrying that name.**
  Rationale: the name is the status strand's stable identity; an older build may have left a duplicate, and removing only the first would leave the band.
- **A failed status-strand removal on the llm arm logs a warning and does not fail `start`.**
  Rationale: same stance as today's failed `ReplaceStrand` — a leftover band costs the operator screen rows, not the run.
- **The llm-arm removal also deletes the status sidecar (`status-strand.json`), best-effort.**
  Rationale: the sidecar records a strand GUID and build for a strand that no longer exists;
  deleting it keeps the scratch tree from claiming a strand that is gone.
  A failed delete logs and continues.
- **`step` (`loomPreStep`, reached by both `lyx loom step` and `lyx shed step`) calls reed `Up` in place of `ensureStatusStrand` and does no strand work.**
  Rationale: `lyx shed step` is what the ly-drive driver runs on every step, so leaving `ensureStatusStrand` in `step` re-adds the status strand on every llm step and defeats the whole task.
  The go driver never steps — it runs detached `lyx loom run` — so `step` has no driving surface of its own to show.
  Keeping `step` driver-blind preserves its existing contract (the driver `seedAndCommitBootstrap` returns stays discarded there) and keeps the driver read confined to `start`.
  `Up` stays because the producers under `step` spawn agents into reed panes, which is why `ensureStatusStrand` called it.
- **`step` does not remove a leftover status strand.**
  Rationale: removal needs the driver, which `step` must not read;
  and an unconditional removal would strip the band from a halted go run an operator is stepping by hand.
  An llm run's leftover band is cleared by the next `start`, which is the resume path.
- **`ensureStatusStrand` keeps its name and body for the go arm; the reed `Up` call moves out of it to its callers.**
  Rationale: its keep/replace/add logic is correct for the go arm unchanged;
  moving `Up` out lets the llm arm and `step` share the substrate call without the strand work.
  The plan writer may restructure helper boundaries as long as the behaviour above holds.
- **`CONSTRAINTS.md`'s Driver Choice Single-Site Invariant is amended in the same commit.**
  The "selects a spawn and nothing else" clause becomes "selects the run's driving surface — which driver spawns and whether the session carries the loom status strand — and nothing else."
  Rationale: removing a status strand is spawn-set selection, not behaviour gating, but the current wording names only the spawn;
  an unamended invariant would read as violated by this change.
  The tripwire test (`driverFieldReadCarveOuts`) needs no entry: the read stays inside `internal/loomcli`.

## Constraints

- Driver Choice Single-Site Invariant (amended as above): the driver is read once, in `start`, and the branch selects the driving surface only.
- Shed Verb-Set Invariant: the change to `step` lives in loom's own `PreStep` hook, never in `internal/shedverbs`.
- CLI/Cobra Invariant: no new command or flag; the `start` long help text is updated where it says the status strand is always ensured.
- Documentation Lifecycle: `docs/overview.md` updated in the same commit.
  No roadmap change: this is a behaviour adjustment, not a planned item.
- The bootstrap lock's hold region in `start` is unchanged: the strand branch stays inside it.
- `step`'s early run-lock probe stays above all substrate work, so a busy step still touches neither reed nor the seed.

## Auto-mode assumptions

- Driver values are the two-value switch `mustUseLLMDriverArm` already defines: `llm` selects the llm arm, anything else (empty included) the go arm, which gets the status strand.
- Seeds are immutable per run (`WriteSeed` refuses a disagreeing seed), so a run never switches driver mid-flight;
  the removal path exists only for sessions carrying a band from an older build or older start.
- The go driver never invokes `step`; `step` callers are ly-drive and operators.
- The hub's `collapsed_rows: 16` is already in place and is the right ly-drive height; nothing in this task touches config.

## Open risks

- An operator who pre-seeds `go` and drives it with bare `lyx loom step` (never `start`) gets no status strand.
  Accepted: `start` is the go run's bootstrap, and `lyx loom status` remains available.
- A running ly-drive session keeps a leftover status band until its next `start`; `step` does not clear it.
  Accepted per the `step` decision above.
- Reed's `RemoveStrand` cascades through descendants.
  The status strand is added as a plain below-parent strand with no children, so the cascade removes only it;
  if some older build parented anything beneath it, that would be removed too.

## Acceptance criteria

- `start` on an `llm`-seeded run over a session with no status strand adds none, and the driver strand is spawned as today.
- `start` on an `llm`-seeded run over a session holding a `loom-status` strand removes it (every same-named strand) before the driver-spawn step, and deletes the status sidecar.
- `start` on an `llm`-seeded run whose status-strand removal fails still proceeds to the driver spawn and logs a warning.
- `start` on a `go`-seeded run keeps today's keep/replace/add behaviour and sidecar write, unchanged.
- `start` on an unseeded worktree (seeded `llm` by `resolveSeedDriver`) adds no status strand.
- Reed `Up` still runs on both arms of `start` before any strand work.
- `lyx loom step` and `lyx shed step` on a loom run bring reed up and add no status strand, whatever the seed's driver.
- `lyx loom step` over a session already holding a status strand leaves it untouched.
- A busy `step` (run lock held) still refuses at the early probe, above seeding and reed.
- `step` does not read the seed's driver (the value from `seedAndCommitBootstrap` stays discarded).
- `TestDriverChoiceSingleSiteInvariant_OnlyLoomcliReadsTheSeedDriverField` passes with no new carve-out.
- The `internal/shedcli` parity tests still pass; comments there naming `ensureStatusStrand` in `step`'s path are corrected.
- `CONSTRAINTS.md` and `docs/overview.md` reflect the new behaviour.

## Notes for the plan writer

- Call sites of `ensureStatusStrand`: `start`'s `RunE` in `internal/loomcli/start.go`, and `loomPreStep` in `internal/loomcli/arm.go`.
  The helper itself is in `internal/loomcli/sharedbootstrap.go`; the name constant, add spec, and `resolveStatusStrandAction` are in `internal/loomcli/bootstrap.go`; the sidecar is in `internal/loomcli/statusbuild.go`.
- `start`'s long help text and its step comments describe step 4 as "ensure the reed substrate and its status strand"; both need rewording.
- The driver-strand probe seam (`driverPaneProbe`) already reads strands and removes a driver strand; a similar seam, or the reed engine directly as `ensureStatusStrand` uses it, serves the llm-arm removal.
  Tests over `start` already fake reed through seams — see `start_watchdog_test.go`, `smoke_starttail_test.go`, `sharedbootstrap_test.go`.
