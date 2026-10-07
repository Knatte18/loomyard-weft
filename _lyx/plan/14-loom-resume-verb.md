# Card 14 — `lyx loom resume`, the verb that only wakes a parked driver

**Create:**
- `internal/loomcli/resume.go`
- `internal/loomcli/resume_test.go`

**Edit:**
- `internal/loomcli/cli.go`
- `internal/loomcli/start.go`
- `internal/loomcli/driverresume.go`
- `internal/loomcli/driverresume_test.go`
- `internal/loomcli/driverlaunch.go`
- `internal/loomcli/wiring.go`
- `contracts/specs/refusal-spec.md`
- `docs/overview.md`

**Uses:**
- `internal/reedengine/directory.go`

**Intent:**
A new loom verb, `lyx loom resume`, run in the task worktree, resumes a halted run only by waking its live, parked driver (D7).
It types the driver resume line and removes the park marker through the same branch `lyx loom start` takes for a parked driver, shared rather than copied.
It never spawns a driver, never creates or attaches a strand, never calls reed's `Up` and never switches a client.
It reads only the run's state, the driver strand's liveness, the park marker and the child's run lock, never the seed's driver value (`PATTERN-driver-choice-single-site`).
Bound: the verb can only deliver the resume line a parked driver already waits for; it cannot start anything that is not there.

Shared branch (in `start.go` and `driverresume.go`): `resumeParkedDriver` takes the calling verb's name and returns the stop-report path beside its error, and `refuseOverUnfinishedMerge` and the driver-not-parked refusal take the verb name too, so each message names the verb that called it as the retry.
`lyx loom start`'s messages stay word for word what they are today, so its existing tests are unchanged; `lyx loom resume` never tells its caller to run `lyx loom start` as the retry.
`driverresume_test.go` gains the row pinning that a resume-called failure names `lyx loom resume` and not `lyx loom start`.

Strand table: the verb reads reed's sessionless `Directory` (card 4), so a session that died with the machine still shows a dead driver strand; `driverlaunch.go`'s reed adapter gains that read behind a narrow seam on `loomCLI`, filled in `wiring.go` beside the existing driver seams.
A driver strand is live when its row is live and not retiring; a retiring strand counts as not live, as `lyx loom start` never adopts one.

The verb takes the bootstrap lock `lyx loom start` takes (`loomengine.LoomBootstrapLock`), blocking until it is free, reads everything only after holding it, and releases it on every path out, so the two verbs never run at once.
Outcomes, decided by the run's state first; every refusal is a JSON error envelope:

- no status file: refused; way forward `lyx loom start` in the task worktree;
- done: a no-op success saying the run is done;
- awaiting: refused; way forward `lyx loom approve` or `lyx loom reject` in the task worktree, after which a batten run watching the child resumes it on the recorded decision, and `lyx batten run <slug>` from the prime when no batten run watches it; it reads no batten state;
- running: a no-op success saying the run is already running when the driver strand is live or the child's run lock is held; otherwise the no-live-driver refusal below;
- halted (blocked, paused, failed) with the child's run lock held, whatever the strand: refused, a driver is still finishing its post-run work; way forward: run `lyx loom resume` again in a few seconds;
- halted, live driver, no park marker: refused with start's existing `driver_not_parked` kind, naming `lyx loom resume` as the retry; the merge is not probed;
- halted, live driver, park marker present: over an unfinished merge refused with start's existing `merge_in_progress` kind (its remedy then re-runs `lyx loom resume`); otherwise resumed through the shared branch, and the success envelope names the stop-report path;
- halted, no live driver: over an unfinished merge the same `merge_in_progress` refusal; otherwise refused, with the way forward told apart by the strand table:
  - a dead driver strand (present, not live, not retiring): both ways with their conditions, `lyx batten run <slug>` from the prime when no batten run is watching the child, which brings the driver back, then `lyx loom resume` again; `lyx loom start` in the task worktree when a batten run is already watching or its revive failed;
  - a retiring driver strand: `lyx loom start` in the task worktree, which removes the retiring strand and spawns a fresh driver;
  - no driver strand in reed state: `lyx loom start` in the task worktree.

A resume line that cannot be delivered is the shared branch's failure, naming `lyx loom status` and a re-run of `lyx loom resume`.
Every refusal names a lyx verb or the retry of `lyx loom resume` as its way forward and no denied command (`PATTERN-refusal-way-forward`, `PATTERN-no-denied-recovery`), and each lands as a row in `refusal-spec.md`'s loom section, prefixed `resume:`, in this commit.
`resume_test.go` reaches every outcome above in one table over the seams (status file, run-lock file, park marker, strand table, merge probe and driver sender as fakes), and asserts for each the envelope, the exit code, that no spawn, add or `Up` was called, and that no message names `lyx loom start` as the retry; one row pins that the verb blocks on a held bootstrap lock and reads state only after it is released.

`cli.go` registers the verb in loom's subtree with a `Short`, a `Long` stating the outcomes above, and an example; it uses the full wiring like `start`, since it needs the driver sender.
`PATTERN-sandbox-coverage` is met as is: the loom module is already covered, and coverage is module-level.
`docs/overview.md` adds `resume` to loom's verb list and one sentence beside the parked-driver sentences saying `lyx loom resume` wakes a parked driver and never spawns one, while `lyx loom start` still resumes as before.

**ImpactSummary:** One new verb; `lyx loom start`'s resume branch gains a verb parameter with its own messages unchanged.
