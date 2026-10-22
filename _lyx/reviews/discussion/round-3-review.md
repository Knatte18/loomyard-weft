---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:107-110 and :263
    summary: "The second gate sits behind batten's own pre-run hooks, which seed the status and clear scratch markers lock-free, so a late run or step still writes status after the cancel removed it and the criterion writing no status is unreachable."
    fix: "In D3 (the paragraph on the second gate, and the sentence naming an unregistered one still short of the run lock, such as lyx batten step in shed's preflight) name the real pre-lock stage: battenPreRun and battenPreStepUnmarked in internal/battencli/arm.go run before shed's Run or Step takes the run lock, probe or skip the lock, and for an absent status call battenshed.ClearRunMarkers and then state.UpdateJSON to seed Worktree-Create running. Add to the pre-drive check's scope a second call site, inside those two seeding closures, so the closure re-reads the cancel marker and the seed and returns its refusal without writing when the marker exists or the seed is gone; the closure runs under the status lock that cancel step 7 holds, and the marker stands until step 9, so no status file is written between the records removal and the marker removal. Keep the check in shed Run and Step for the resume path. Rewrite the acceptance criterion at line 263 to say the step or run refuses in the pre-run hook or at the run lock, writing no status, no seed and no claim, and add that a start after the marker's removal meets the missing seed before any status write."
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:143-144 and :289
    summary: "Reading the tackler marker after the end misses a tackler strand whose driver was ended between Shuttle.Start and the marker write, and the record claims a still-live batten's tackler is always read."
    fix: "In D4 (the paragraph ending with the read follows the end and the lock take) and in the plan-writer note at line 289, state that sheddriver's tackler call writes the attempt record and the in-flight marker only after Shuttle.Start returns (internal/sheddriver/tackler.go, the run.StrandGUID call and recordStartedTackler), so a driver ended inside that window leaves a live tackler strand no marker or attempt record names. Either name how the cancel finds such a strand (the research done and its answer recorded, for example listing the prime session's live strands through shuttle and stopping the one spawned for the tackler role), or add it to Open risks with its bound, which is the tackler's deadline from its settings. Reword the sentence so it claims the marker covers a tackler the driver had recorded, not every tackler a still-live batten spawned."
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:147 and :267
    summary: "On Windows runproc.End returns nil with nothing signalled, so no survivors error can arise and the stated refusal for a survivor after the end has no mechanism; a process registered after the step 4 list is neither named nor refused."
    fix: "In D4 give the Windows phase an explicit re-list: after the lock takes of step 2 and before the strand stop and removal, the phase calls runproc.List for the slug again, applies the same creation-time filter, and refuses with each pid, role and taskkill line when any record remains, leaving the marker as step 5 says for a run that survives. State that on Linux the survivors error from End does this work. Then reword the survivor sentence at line 147 and the criterion at line 267 so the Windows survivor refusal is produced by that re-list, not by End. If the re-list is not wanted, delete the Windows survivor clause from line 147 and line 267 and state in D4 that a process registered between the step 4 list and the lock takes is covered only by the lock-holder refusal."
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:169-174 and :140
    summary: "A cancel without --remove over a pair whose task worktree was already gone leaves the weft worktree or branch, and the later start is not refused by the pre-run check, so fabric's add refuses bare or adopts the old records branch and the run is not halted with the precondition prefix."
    fix: "In D5 change the start's pre-run refusal from the task worktree is present to any local part of the pair being left that Add would trip on or adopt: the task worktree, the weft worktree (fabricengine.RecordsWorktreePath) or a weft branch, keyed on the same pair-state read the removal uses to decide some part is left; keep the task-branch-only case to D11's Worktree-Create halt. Name in the refusal lyx batten cancel <slug> --remove. Add an acceptance criterion for a cancel without --remove over a slug whose task worktree was gone with the weft side left, followed by lyx batten run, which refuses in the pre-run with no seed and no claim. Alternatively extend D11 so the add's weft worktree directory already exists refusal (internal/fabricengine/add.go, weftTarget check) maps to the precondition prefix with the same way forward, and say the weft branch case is adopted and why that is accepted."
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:67
    summary: "The nothing-left refusal names lyx board list as its way forward, but without --remove it ignores a left task branch, so after lyx fabric remove of a kept pair the refusal does not clear the slug."
    fix: "In D2's last refusal bullet, add that when no --remove was given and a task branch of the slug is left locally (a local ref read, no network call), the refusal names lyx batten cancel <slug> --remove as its way forward instead of lyx board list. Add that wording to the acceptance criterion at line 274 as the refusal a plain cancel meets in that state."
---

### [BLOCKING] Second gate is behind the hooks that already write status

**Location:** `_lyx/discussion/decision-record.md:107-110`, `:263`; `internal/battencli/arm.go` `battenPreRun` and `battenPreStepUnmarked`, `internal/shedverbs/run.go:31`, `internal/shedverbs/step.go:367`.

**Issue:** The record says an unregistered `lyx batten step` still in shed's preflight is refused by shed's busy refusal or by the pre-drive check, "writing nothing".
Shed's `preflight` only validates and creates lock parent directories (`internal/shedengine/run.go:118`).
Before it, `shedverbs` calls batten's `PreRun` or `PreStep` hook, and both hooks, for an absent status, call `battenshed.ClearRunMarkers` and then `state.UpdateJSON` to seed `Worktree-Create` running.
`battenPreStepUnmarked` only probes the run lock and releases it, and `battenPreRun` does not touch it, so neither holds the run lock while it writes.
So the busy refusal fires only while the cancel holds the run lock (steps from the hook's lock take to step 9).
Two windows remain: between the marker write and the lock take, and between step 9's lock release and the marker removal.
A late `run` or `step` that passed `armAt` before the marker also writes its status after step 7 removed the run directory, and the second gate then refuses, leaving a stray status file with no seed.
A later `lyx batten run` finds that status, takes the resume branch, and skips D5's start-only refusal of a present task worktree, so a kept pair is adopted with its old child records.
Round 2's F1 asked for the second gate to stop exactly this reseed, and the gate as placed does not.
Answer to the focus question: a batten past the marker check is refused or ended before the locks are released only at the run lock, not at the status write, and the record names the wrong stage for it.

### [MEDIUM] Tackler strand started but not yet recorded is not read

**Location:** `_lyx/discussion/decision-record.md:143-144`, `:289`; `internal/sheddriver/tackler.go:247-266`.

**Issue:** The record places the marker read after `End` and the lock take, which is the right place relative to step 7's scratch removal and settles the ordering question: a tackler recorded before the end is read, and none is spawned after it.
The claim that a tackler "a still-live batten spawned before the end is read" is wider than the marker.
The tackler call starts the strand with `Shuttle.Start`, and only then writes the attempt record and the in-flight marker (`recordStartedTackler`).
A driver ended by `runproc.End` inside that window leaves a live tackler strand that neither file names, so the cancel never stops it and it runs to its deadline in the prime's session.
Open risks list neither the gap nor a way to find the strand.

### [MEDIUM] Windows survivor refusal has no producer

**Location:** `_lyx/discussion/decision-record.md:147`, `:267`; `internal/runproc/end.go:110-112`, `internal/pairteardown/teardown.go:268-300`.

**Issue:** The marker is written before the Windows list (D3 step 4), so the ordering question is settled.
The reused-pid bounds are stated: the cancel marker's liveness is a lock its writer holds, and a registered record is dropped when its process was created after its registration, with the clock-moved-back bound named.
What fails is the survivor path.
On Windows `End` returns `nil, nil` before any signal, so `endOwnedBeforeSession` never receives a `SurvivorsError` and never refuses with pids and roles.
The only Windows process check is the one pre-`End` list at step 4, so a process registered after that list (including a batten that passed the marker read and registers later) is neither named nor refused, while the record and its criterion promise a survivor refusal with pid and role.

### [MEDIUM] Weft-side leftovers after a plain cancel are not covered by the start refusal

**Location:** `_lyx/discussion/decision-record.md:169-174`, `:140`; `internal/fabricengine/add.go:216-219` and `:180`, `internal/battencli/wire.go:812-830`.

**Issue:** D4 names the state task worktree gone with a reed session or the weft side left, and the plain cancel keeps what is left.
D5's pre-run refusal and D11's mapping cover a present task worktree and the task-branch refusals only.
A later `lyx batten run` therefore passes the pre-run, reaches fabric's add, and meets the bare weft worktree directory already exists refusal (not `*ErrBranchExists` or `*ErrRemoteLeftover`), which Worktree-Create does not map to the precondition prefix, so the blocked stop goes to the tackler row the record elsewhere closes.
A left weft branch with no worktree is adopted by the add, so the new run inherits the old run's records branch.
The way forward D5 names, `lyx batten cancel <slug> --remove`, would clear the slug, but nothing steers the start to it in this state.

### [LOW] Nothing-left refusal does not name `--remove` for a left task branch

**Location:** `_lyx/discussion/decision-record.md:67`.

**Issue:** After a plain cancel and `lyx fabric remove`, the slug is blocked by a task branch.
A plain `lyx batten cancel <slug>` there counts nothing left (branches count only with `--remove`) and names `lyx board list`, which does not clear the slug.
The branch read needed to name `--remove` is local, so no network call is added.

## Rejected

- Focus 3, a `--remove` re-run after step 6 or on a run that never had a pair: not a defect. D5 runs `RemovePair` only while some part of the pair is left, reads an `ErrPairNotFound` wrap as done (`internal/fabricengine/remove.go:606-639` returns it only when no worktree, branch, launcher entry or origin branch remains), and goes on to the records removal, claim clear, lift, notice and marker removal. The snapshot tolerates absent worktrees (`SnapshotUncommitted` guards each by presence).
- Focus 2, where the tackler marker read sits relative to `End` and the scratch removal: stated in D3 step 5, D4 and the plan-writer note, and consistent; the only gap is F2.
- Focus 4, ordering: the Windows list runs after the marker is written and a fresh marker is removed on that refusal; stated and consistent. `RegisteredAt` carries nanoseconds, so the creation-time comparison does not truncate.
- Focus 5, kept pair whose entry the caller marked done, and a later `lyx batten run` on a kept pair with the task worktree present: both reach the refusal or removal the record names, and the named way forward clears the slug.
- A late `lyx batten goto`: `shedengine.Goto` refuses an absent status and never seeds one, so the second gate needs no goto call.
