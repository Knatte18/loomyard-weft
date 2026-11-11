# An upkeep pass runs under a slot it yields to a gate, side by side where tests declare it, and writes per round

`testmap-upkeep-runs` · Task · feature, testkit, gate, dobby · running · Describe

Decision record: [Sketch](decision-testmap-upkeep-runs.md)

Filed from the Sketch `testmap-upkeep-design`.

Takes: testmap-upkeep-plan's plan, queue, job and exit record; gateslot's unexported tryAcquireSlot, the pool's drain and gateslot.RunRecord
Delivers: the verb side of D8: the hidden --upkeep flag refusing package arguments, its own RoleTestMap registration under the hub owner, refusal beside a live pass, exit before the plan on a failed registration, the exit record read and written only under --upkeep, HEAD read before each job and wake, queued work leaving out dirty-skipped, failed-at-HEAD and spawn-marked work; the take only while two slots are free and no drain waits, the yield that kills runs, writes finished entries and releases the slot and lock, the wait at most once a second with backoff, the lock retry and fresh plan on retake, no slot on a hub below two, and hand calls' blocking take, lock wait and whole rebuild (D9); side-by-side runs for top-level t.Parallel() tests up to the tier-0 parallel, per-round writes, the per-round HEAD and dirty read, signal handling and the job order from gate run records (D10)
Exports: lym gate testmap --upkeep, a registered pass that exits on a failed registration, which testmap-upkeep-dobby starts; gateslot's exported non-blocking take

The part changes these packages:

- internal/gatecli (about 700 changed lines)
- internal/gateslot (about 50 changed lines)
