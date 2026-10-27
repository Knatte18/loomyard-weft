# Support log — dobby-and-adopt-hardening

## Interview

Autonomous run with no advisors: every question below was settled by the writer and is an auto-pick in the question ledger.

### Turn 1 — exploration

Read the board entry, the board list (no `Decision for dobby-and-adopt-hardening:` note stands; `test-logs-leak-into-bolt` touches trace logs under `_board/.lym/logs`, and `workspaces` is the parent project), `lym loom peers` (no landed parent commit, no peer touching these packages).
Read `internal/dobbyengine` (doc, `Run`, `Live`, `SpawnDetached`, the tidy count), `runDobby` in `internal/beamcli`, `internal/selfexec`, `internal/lock`, `DobbyLandings` and the lock probes in `internal/battencli`, the adopt suite spawn in `internal/adoptcli`, the gate keys in `internal/gateslot` and the hub's `gate.yaml`, the logger doc, and a batten `status.json` whose Land Done entry has an empty output and no machine.

### Turn 2 — part 1, the re-exec exit

Question: fix by grace only, or keep the lock across the exec?
Recommended and picked: drop the bare pre-check, take the lock in `Run` with the grace, register after the lock, and give a re-exec a ten-second wait (Decisions 1 and 2).
Question: what is posted to the orch?
Picked: a failed exec and a refused re-exec start; a refused fresh start is the idempotent `EnsureLive` case and posts nothing (Decision 3).

### Turn 3 — part 2, the log

Question: `internal/logger` or a fixed bounded file?
Picked: the fixed file, written per line by the log seam and rotated to `.1` at 1 MiB, with the process's raw stdout and stderr moved to `dobby.out` so rename-rotation works (Decision 5).
Question: what is a change?
Picked: the entry's list, with a failing pass logged once per episode rather than per retry, and a ten-minute heartbeat at a package constant (Decision 6).

### Turn 4 — part 3, the tidy count

Question: a shared tidied mark on Bolt, or counting by landing machine?
Picked: record the landing machine on the history entry and count only this machine's landings (Decisions 7 and 8).

### Turn 5 — part 4, the adopt check

Question: how to report per tier?
Picked: a hidden `lym adopt check` that runs the gate in-process with a per-tier observer, writes `check.json`, and posts through creel; `lym beam status` reads the record (Decisions 10 and 11).
Question: what tiers does `adopt` run on this hub?
Picked: the reconcile migration copies a tuned `finalize` value into a missing `adopt` key (Decision 9).

## Rejected alternatives

- Keeping the dobby lock held across the exec by clearing close-on-exec on its fd and handing the fd to the new image: removes the gap entirely, but `gofrs/flock` cannot adopt an fd, Windows has no exec, and the graced acquisition already defeats every momentary prober.
- Making the `runDobby` pre-check graced instead of dropping it: leaves two acquisitions and a second gap between them, and keeps registration before the real lock.
- Logging the dobby through `internal/logger`'s trace files: a trace file is per process and trace id, goes silent at its 8 MiB cap, the dobby has none today because it runs from the hub root, and the operator reads `dobby.log` at its fixed place.
- Rotating `dobby.log` while it is the process's stdout and stderr: writes follow the renamed inode, so the bound would not hold.
- A shared tidied mark written to Bolt or the run records: needs the dobby to commit and push, and races between machines; counting by landing machine gives the same "tidied once" result with a read only.
- Baselining the landing count at `lym adopt`: records pulled while this machine's dobby was not running would still be counted at its next start.
- A `--notify` flag on `lym gate test`: makes the gate module post to the orch and know the adopt check's record.
- Parsing `TierSummaryLine` from a child `lym gate test`'s stderr: a second parser of the gate's output.
- An `adopt` template value of tiers 0, 1 and 2: the template declares no tiers above 0, so the hub's tuned `finalize` value is copied instead.
- Making the heartbeat interval a `beam.yaml` key: no one asked for it to vary.

## Review rounds

- Round 1, APPROVED with three MEDIUM findings and twelve LOW or NIT ones, all fixed in the record.
  The exits before `Run` gained their lines (Decision 3), the adopt check gained a lock of its own and a hub reconcile before it reads its key (Decision 10), and the re-exec and dead-check tests gained start ticks beside the pid (Decisions 2, 11).
  The fresh dobby's unregistered window was bounded rather than closed by moving the wiring after the lock, and a dobby that escapes a teardown now exits at a probe tick that finds its worktree gone (Decision 1).
  The fixer's self-check made the adopt check fail on a failed reconcile rather than warn and go on, and made a record without start ticks read as a re-exec for the first deploy of this change.
- Round 2, APPROVED with six MEDIUM findings and seven LOW or NIT ones, all fixed in the record.
  A `placeDobby` refusal after the worktree resolves and a fatal `Run` error after a re-exec gained their line and post (Decision 3), and the worktree-gone exit writes to the prime's log and stops a tidy in flight (Decision 1).
  `recordStart` and every pid writer share the start-ticks re-exec test (Decision 2), and the dobby moves inherited fds onto `dobby.out`, which the lock holder rotates where `selfexec` execs and a spawner that reads the lock free rotates on Windows (Decision 5).
  The adopt check's own output goes to `check.out` (Decision 10).
  Foreign landings already queued are cleared by a one-time emptying of `LandedSinceTidy` rather than a per-count drop, since run cleanup removes run directories (Decision 8).
- Round 3, APPROVED with one MEDIUM finding and two LOW ones, all fixed in the record.
  The worktree-gone exit tests the worktree's `.git` link at every loop iteration, since the dobby's own state access recreates a removed root, and keeps the tidy of the last tick that ended with the link present (Decision 1).
  The re-exec test is named once, counting a record without start ticks as a re-exec for the lock wait and the notices (Decision 2), and the Windows spawner's `dobby.out` rotation is named as the one exception to the lock-holder rule (Decision 5).

## Question ledger

- Fix for the re-exec exit — auto-pick: drop the pre-check, graced lock in `Run`, register after the lock, ten-second wait for a re-exec; resolved (Decisions 1, 2).
- Orch notices for dobby exits — auto-pick: failed exec, refused re-exec start, and a wiring failure, a `placeDobby` refusal or a fatal `Run` error after a re-exec only; resolved (Decision 3).
- Pre-exec log line — auto-pick: a callback form of `selfexec.Watcher.Check`; resolved (Decision 3).
- Log mechanism — auto-pick: fixed `dobby.log` written by the seam, 1 MiB rotation to one previous file, raw output to `dobby.out`; resolved (Decision 5).
- Logged events and heartbeat — auto-pick: the entry's list, failing pass once per episode, ten-minute heartbeat as a constant; resolved (Decision 6).
- Tidy count per owner — auto-pick: landing machine on the history entry, count only this machine's, unmarked entries never counted; resolved (Decisions 7, 8).
- Adopt gate key tiers — auto-pick: template tier 0 scope all, reconcile copies a tuned `finalize`; resolved (Decision 9).
- Adopt progress mechanism — auto-pick: hidden `lym adopt check`, in-process gate with observer, `check.json`, creel notices; resolved (Decision 10).
- Binary-replacing test tier — auto-pick: tier1 with the prebuilt lym, since `selfexec` never execs under `go test`; resolved (Constraints).
