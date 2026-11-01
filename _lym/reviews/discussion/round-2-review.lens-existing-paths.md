```yaml
lens: existing-paths
outcome: reported
```

**Lens: existing-paths.** I traced the existing code paths that write, end or read what the record introduces or changes.

**Paths traced, per decision**

- **D3, runner ended by `runproc` callers.** Every caller below ends a runner registered under the task slug with `runproc.RoleGate`, because none of them spares that role. Each end gets D3's terminating-signal handling, or the no-exit-file supersede if the 5 s `endGrace` runs out.
  - Pair teardown: cancel `internal/pairteardown/cancel.go:156` and the request teardown at `teardown.go:299` (spares nothing), skip at `skip.go:81` (spares only `RoleDobby`). **Survives.**
  - `lym tender pause` and reap, `runproc.End` at `internal/tendercli/deps.go:113` and `deps.go:321`. Matches D3's "held agent". **Survives.**
  - `lym fabric anchor --set`, at `internal/fabriccli/anchor.go:411`. It calls `EndSparing` over the worktree name and spares only push and sync (`anchordeps.go:77-82`), so a move also ends the runner. That is harmless: the move also changes `gateslot.WaitDir(location.AnchorPath())`, so later fronts resolve a fresh key tree and start a new run, and the old runner's marked exit file reaches no one. **Survives**, though D3's list of ending paths does not name it.
- **D3, tender reap.** `orphanOwner` ends a slug no worktree carries. `unregisteredOrphans` only takes a process whose working directory was deleted, outside every worktree (`internal/tenderengine/processes.go:116-150`), so the unregistered runner in the prime is not reaped while its tree exists. **Survives.**
- **D3, a killed runner's tiers.** Tiers start through `proc.StartTied`, which sets `Setpgid` and `Pdeathsig=SIGKILL` (`internal/proc/tied_linux.go:18-19`, `internal/gatecli/child.go:38`). A runner killed by SIGKILL or OOM takes its direct tier children with it, so D3's "frees its slots" claim holds. **Survives.**
- **D1, slot wait records.** The runner's `acquireSlot` writes its wait record with its own pid into `WaitDir(anchor)` (`internal/gatecli/run.go:541`). That record is read by `loomcli/reviewstatus.go:179` (`gateWaiting`) and `hubgeom/webstergeom.go:42`, and still describes a real wait. **Survives.** I did not read `gateslot.ReadWaits` (`wait.go:62`), so whether it tolerates D1's per-key subdirectories inside the same directory is unverified.
- **D1, `slot_free` holders.** These are read from the pool, as `answerSlotBusy` does today. **Survives.**
- **D2, slot-wait source.** `slotWait` is set only to `shuttleengine.DefaultAwaitCap` (`internal/gatecli/rungate.go:29`, `cli.go:92`), and `cli_wait_sec` is retired (`internal/gateslot/doc.go:51`). No other knob competes with D2 or D4. **Survives.**
- **D5, `ReadRunRecords`.** It skips directories and non-`.json` files (`internal/gateslot/runlog.go:128-153`), so the new fields and the key directories do not disturb it. `pruneRunLog`'s 14-day cutoff only drops old calls. **Survives.**
- **D5 and D3, `lym gate report` totals (`internal/gatecli/report.go`).** **Breaks**, see F1.
- **D6, Go verbs that run card lines.** These go through `cardverify.Runner` (`internal/websterengine/cardverify.go`, `recordbatch.go`). With the caller's context passed in and the strand removed, they still run their `--wait` lines in-process. **Survives.**
- **D7, Go verify sites.** They keep `gateslot.Config.GateCall`'s `--wait` form. **Survives.**
- **Constraints, the strand removed for every `verifyrun.Run` caller.** The only production callers are `cardverify` and `battencli` (`pattern/PATTERN-gate-slot.md`, Enforcement). **Survives.**

**Findings**

- **F1: MEDIUM, design, D5 and D3 (decision-record.md lines 132 and 187).** `lym gate report` will count abandoned tiers as failures.
  - D3 adds frequent new cancel sources: a passed lease, a supersede on every tree change between calls, and a signal to the runner. `execution.report` writes the tier those cuts land in as an ordinary record with its exit code (`internal/gatecli/run.go:479-496`), and `report.go:147-148` totals every record.
  - D5 adds `Abandoned`, but the Constraints only say records without it "still total in `lym gate report`". The record never says whether `report` leaves out or separates records carrying `Abandoned`.
  - Result: per-key pass/fail totals and wall times in `lym gate report` absorb every lease-, supersede- and signal-cut tier as a real outcome. D5 marks the state but the existing reader still misreads it.

**Tool calls**

1. Grepped `slotWait` and `cli_wait_sec`, and the callers of `runproc.End`, `EndSparing` and `EndRole`: the manifest had no caller of the end functions or source of the slot wait.
2. Read the `anchordeps.go` end, the pair-teardown end, the `tendercli` end contexts and tender's orphan grep: to see which roles each existing end spares.
3. Read tender's `orphanOwner` and `unregisteredOrphans` (with dobby-owner and spare-list greps): to judge whether reap ends an unregistered runner in the prime.
4. Read anchor's `EndDobbies` call and the pair-teardown `endOwned` callers: to get the spare list each end passes.
5. Read `dobbyOwner` and anchor's dobby list: to confirm the anchor end's owner is the task slug.
6. Grepped `gateslot` wait-directory readers: to find existing readers of the directory D1 writes into.
