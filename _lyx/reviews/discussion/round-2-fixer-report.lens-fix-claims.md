```yaml
lens: fix-claims
outcome: reported
```

Lens: fix-claims. I checked each claim the fix added or rewrote against the code at HEAD.

**MEDIUM: the claim that `lyx fabric add &lt;slug&gt; --adopt-local` "wires at the target's anchor"**
- Changed text: the way forward for D10's checkout refusal, "work on the target branch in a pair of its own, which wires at the target's anchor, such as `lyx fabric add &lt;slug&gt; --adopt-local` over a local branch of the slug".
- Summary: nothing in the record makes `Add` wire at the adopted branch's own marker, and D5 says it wires at the parent's anchor.
- Evidence:
  - D5 in the changed record says "`Add` takes the new pair's anchor from the fork point, the parent's Location, or from `--anchor` (D6), never from a read of the not-yet-written weft".
  - `internal/fabriccli/fabric.go:165` describes `--adopt-local` only as "make the pair over a local branch of the slug that already exists".
  - Under D5, an adopt-local `Add` over a branch whose marker differs from the parent's anchor wires at the parent's anchor. The pair's junctions would then not match where its content sits. That is the mismatch the checkout refusal exists to prevent.
  - D5's "An adopt of such a branch writes and commits that value" covers only markerless branches.

**Claims that hold:**
- **Weft write lock path.** `.weft/weft.write.lock` matches `internal/fabricengine/commit.go:195`.
- **Creel state.** It lives under `&lt;hub&gt;/_board/.lyx/creel`. `HubScratchDir` is `BoardDir/.lyx` (`junctionnames.go:167`) and `CreelDirName = "creel"` (`internal/lyxdirs/dirs.go:26`). The daemon is started from the hub directory (`internal/creelcli/spawn.go`).
- **Hibernate cancel.** `lyx hibernate cancel` "stops its daemon" (`internal/hibernatecli/cli.go:102-106`).
- **Freed lock.** "A killed run's parent lock is freed with it" matches the parentlock doc: "a crashed owner frees it at once".
- **Queuing behind a holder.** This matches `Acquire`'s ticket queue in the parentlock doc.
- **A queued Finalize does nothing.** "A child's Finalize that reaches the lock meanwhile … commits no parent records and merges nothing" holds. `internal/landingshed/finalize.go:253-264` takes the parent lock before the sync, the merge and `commitParentRecords` (`:295`). `parentguard.ensure` waits in `Acquire` with no time limit and returns only once it holds the lock, on a pause or on an unreadable record.
- **A spawned pass counts as a dobby child.** `spawnPass` starts the pass with `cmd.Start` from the dobby process (`internal/beamcli/dobbydeps.go:464-485`), so the process table shows it as a child of the dobby.
- **`lyx fabric commit` exists.** See `internal/fabriccli/weft_verbs.go:187`.
- **The `--adopt-local` flag exists.** Only what the claim says it does is in question, as in the MEDIUM finding.

Tool calls beyond context:
1. Grepped `CreelDirName`, the `fabric commit` verb and Finalize's lock and records calls. My context lacked the creel dir constant, the commit verb's existence and the Finalize order.
2. Read `internal/landingshed/finalize.go:250-300` and `internal/dobbyengine/doc.go:30-40`. This confirmed the lock comes before the parent-records commit; my context lacked both.
3. Read `internal/landingshed/parentguard.go:25-130`. This checked that a Finalize waiting on the lock queues and does not go ahead; my context lacked it.
