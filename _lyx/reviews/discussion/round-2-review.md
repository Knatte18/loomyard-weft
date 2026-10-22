---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D3 (cancel marker paragraph) and D4 step 2
    summary: A batten that passed the marker check unregistered and lock-free, such as lyx batten step or run --window, survives the cancel and reseeds and reclaims the run once step 9 releases the locks and removes the marker
    fix: In D3, add a second marker gate placed after the run lock is taken. Batten's shed run path re-reads the cancel marker, and the seed's existence, once it holds shedrun.RunLock and before it reads or writes status. A marker present, or a seed gone, refuses with D3's way forward and writes nothing. Alternatively, register every armAt verb that can drive the run (run, step, goto, the window opener) before the marker check, so step 5's End reaches it. Name the chosen gate in D3. Name the step case in the acceptance criterion about starts during a cancel.
    origin: lens:bound-breaker
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D3 step 3 and D4 (tackler strand stop)
    summary: The tackler marker is read at step 3, before End, so a tackler a still-live batten spawns between step 3 and step 5 is never stopped, and step 7 deletes its marker
    fix: In D3, move the tackler-marker read from step 3 to a point after step 5's End and D4 step 2's lock take, and before step 7 empties the scratch. State in D4 that the strand it names is stopped after that read.
    origin: lens:existing-paths
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md D5 and D3 (each step a no-op when already done)
    summary: A --remove re-run after step 6, or --remove on a run that never had a pair, hands RemovePair a pair of which nothing remains; Remove returns ErrPairNotFound, so the cancel stops at step 6 on every re-run
    fix: In D5, state that step 6 runs RemovePair only while some part of the pair is left (task or weft worktree, warp or weft branch locally or on origin), and that ErrPairNotFound reads as done. State that the task-branch archive-and-delete still runs when only the branch is left. Add the --remove re-run after step 6 to the interrupted-cancel acceptance criterion.
    origin: lens:walkthrough
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D2 (present and absent) against D4 (pair present and no pair, step 6)
    summary: D2 splits on the task worktree's presence and D4 on the pair's, so for a pair whose task worktree is gone but whose reed session or weft side remains it is unsaid whether the cancel phase stops strands and ends the session
    fix: In D4, key the split on the task worktree as D2 does. State that with the task worktree gone the cancel phase still runs its steps 3 and 6, stopping the session's strands and ending the reed session by name. Steps 4 and 5 are skipped, as EndSession skips the loop seizure and run-lock wait for a gone worktree. State that only a slug with no reed session and no pair part runs steps 1 and 2 alone.
    origin: handler
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D3 and D4 (Windows), pid-only liveness
    summary: On Windows no start time is read, so a dead cancel's marker, or a dead registered process, whose pid the OS reused reads alive forever, and D4's taskkill line then names a foreign process tree
    fix: In D3, bound the marker's writer liveness on Windows by the marker's age against the cancel's own longest bound, or by the writer's executable image, and name the way forward for a stale marker. In D4, list Windows records whose pid now runs another image as gone, and never print a taskkill line for them. State both bounds.
    origin: lens:bound-breaker
  - id: F6
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D3 (start race) and D4 (Windows list at step 3)
    summary: On Windows, D4 lists live processes at step 3 before the marker is written at step 4, so a batten that registers between them is neither refused nor ended, since End signals nothing there
    fix: In D4, move the Windows runproc.List refusal to after step 4 writes the marker, so any batten registered before the marker is listed and refused. Its refusal removes the marker it wrote, so nothing is left changed.
    origin: lens:interactions
  - id: F7
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D5 (kept-pair rerun text)
    summary: D5 says a lyx batten run after a kept-pair cancel refuses at Worktree-Create, but the start writes a seed and the claim, and the plain Stuck goes to a tackler, which repeats the 2026-10-20 symptom
    fix: In D5, replace the sentence with the real path. Then either add the kept pair's existing task branch to the pre-run refusal next to D10's dirty check, naming lyx batten cancel <slug> --remove, or map Worktree-Create's existing-branch error to D11's precondition prefix with that way forward. Add the case to the acceptance criteria.
    origin: lens:interactions
  - id: F8
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D2 (ended refusal) against D7 (caller's own status step)
    summary: If the caller marks a kept pair's entry done, as D7 invites, a later cancel --remove refuses as ended and names lyx cleanup --apply, whose reconcile tears down only runs with a batten record, which the first cancel removed; the pair cannot be removed
    fix: In D2, have the ended refusal fire only when batten's run record is still present here, the case cleanup's reconcile tears down. An ended entry with no batten record goes on to the removal. Add the case to the acceptance criteria.
    origin: lens:existing-paths
  - id: F9
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D2 (nothing is left to cancel)
    summary: After a kept-pair cancel and a lyx fabric remove, the unlanded task branch is kept locally and on origin; the rerun refuses on it, and cancel --remove refuses as nothing to cancel, so no named way forward clears the slug
    fix: In D2, count a local or origin task branch of the slug as something left to cancel when --remove is given. In D5, state that such a cancel runs only the task-branch archive-and-delete.
    origin: lens:existing-paths
  - id: F10
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D3 (marker read before armSeed)
    summary: armAt runs the start owner check, a Bolt pull and the claimPrimeOwner write, before armSeed, so a start during a cancel pulls and may write before it meets the marker, unlike D10's ordering
    fix: In D3, place the marker check in armAt ahead of the start owner check, beside D10's dirty check, before the board pull and the claim.
    origin: lens:walkthrough
  - id: F11
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D3 step 5 against D2 and D9 (end-only)
    summary: D3 says an unreachable origin stops the cancel after step 5, yet D2 and D9 have the end-only cancel release locks and post its notice, which are step 9 actions
    fix: In D3 step 5, say that an end-only cancel then runs only step 9's lock release and notice, and leaves the lift, the records push and the marker removal.
    origin: lens:interactions
  - id: F12
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D4 step 2
    summary: A held batten lock is not always a batten started meanwhile, since lyx batten cleanup try-acquires the same run lock, so the survivor refusal names no pid and a ps search finds nothing
    fix: In D4 step 2, word the held-lock refusal as the held lock's path with lyx batten cleanup among its possible holders, and the cancel again as the way forward.
    origin: lens:bound-breaker
  - id: F13
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D5 (task branch archive tag)
    summary: D5 archives one task-branch tip and then deletes the branch locally and on origin; when the two tips differ and neither contains the other, one side's commits go with no archive tag
    fix: In D5, archive the local tip and the observed origin tip, each under its own archive/<slug>/<tip> tag, unless one contains the other. State that the bound covers both.
    origin: lens:bound-breaker
  - id: F14
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D5 (ordinary origin steps)
    summary: D5 says the weft branch is removed on origin as every origin removal does, but Remove deletes it there only with remote set, and D5 never names Remote on the request
    fix: In D5, state that RemovePair runs with Remote set alongside Force and Snapshot.
    origin: lens:walkthrough
---

### [BLOCKING] An unregistered batten that passed the marker check outlives the cancel

**Location:** `_lyx/discussion/decision-record.md` D3 and D4 step 2.

**Issue:** D3 says a batten that passed the marker check before step 4 is either a registered process, which `End` reaches, or a holder of batten's locks.
The code breaks this:

- `registerSeededRun` registers only `run` without `--window` (`internal/battencli/arm.go:291-293`), so `lyx batten step` is never registered.
- Shed takes the run lock only after its preflight, by try-acquire (`internal/shedengine/run.go:518-526`).

A `step` that passed `armAt` before the marker spends the cancel in its preflight, holding nothing.
`End` misses it, and D4 step 2's hook finds the locks free.
The cancel removes the records, clears the claim, releases its locks and removes the marker.
The step then takes the free run lock and drives the run again from the entry row, which rewrites status and reclaims the board entry.
This breaks D3's guarantee and the acceptance criterion that a batten already past the check is ended or refused before anything is removed.

### [MEDIUM] The tackler marker is read before the end

**Location:** `_lyx/discussion/decision-record.md` D3 step 3 and D4.

**Issue:** D3 step 3 reads `shedrun.TacklerMarker` before step 4 writes the cancel marker and step 5 runs `End`, and step 5 can wait up to the run-lock bound for the write locks.
A live batten that meets a blocked stop in that window spawns a tackler and writes a new marker.
The cancel never reads that marker, step 7 deletes it with the scratch (`internal/shedrun/paths.go:283-285`), and the tackler strand lives on.
After step 9 removes the cancel marker, nothing stops that tackler's `lyx batten run` from reseeding the run.

### [MEDIUM] A --remove re-run after step 6 always stops at step 6

**Location:** `_lyx/discussion/decision-record.md` D5 and D3.

**Issue:** `Remove` returns an error wrapping `ErrPairNotFound` when nothing of the pair is left (`internal/fabricengine/remove.go:153`), and `RemovePair` passes it through (`internal/pairteardown/teardown.go:369-373`).
D5 words every `RemovePair` refusal as its own way forward, then the cancel with `--remove` again.
So a `--remove` cancel interrupted after step 6 never reaches the records removal, claim clear, lift, notice or marker removal on its re-run.
The same holds for `--remove` typed on the 2026-10-20 case, where no pair ever existed.
D3's promise that each step is a no-op once done does not hold here, and neither does the interrupted-cancel criterion.

### [MEDIUM] Present and absent split differently in D2 and D4

**Location:** `_lyx/discussion/decision-record.md` D2 and D4.

**Issue:** D2 splits on whether the task worktree is present.
D4 splits on "the pair present" against "no pair", yet D4 step 6 ends the session "by name when the task worktree is gone", which means the phase runs with the task worktree gone.
Take a pair whose task worktree was removed by hand while its reed session still stands.
If it counts as no pair, the verb runs only `End` and the lock take, so the session and its strands survive and reed relaunches them, the 2026-10-21 symptom.
`EndSession` already handles a gone task worktree (`internal/pairteardown/teardown.go:206-263`), so the split should follow it.

### [MEDIUM] Windows liveness by pid alone

**Location:** `_lyx/discussion/decision-record.md` D3 and D4.

**Issue:** On Windows a record with an empty start time lives while its pid does (`internal/runproc/doc.go:11`, `:24`), and D3's marker records a start time the platform cannot read.
A cancel killed mid-way whose pid is later reused reads as a live writer.
Every start and every cancel then refuses with "wait for the cancel to end", and that never clears.
D4's refusal for a reused registered pid prints `taskkill /PID <pid> /T /F` for whatever process now holds the pid.
Following that way forward kills a foreign process tree.
The record states no bound for either case.

### [MEDIUM] The Windows list runs before the marker exists

**Location:** `_lyx/discussion/decision-record.md` D3 and D4.

**Issue:** D4 lists the slug's live processes on Windows at D3 step 3, and the marker is written at step 4.
`End` signals nothing on Windows (`internal/runproc/end.go:110-112`).
A `lyx batten run` that registers in `armSeed` between step 3 and step 4 (`internal/battencli/arm.go:248-250`) is neither listed nor ended.
D4 step 2's hook catches it only when it already holds a batten lock.

### [MEDIUM] The kept-pair rerun goes to a tackler, not a refusal

**Location:** `_lyx/discussion/decision-record.md` D5.

**Issue:** D5 says a later `lyx batten run <slug>` "refuses at Worktree-Create on the existing branch".
On a clean source the start passes D10's check, writes the seed and the board claim, and Worktree-Create returns a plain Stuck carrying the add's error (`internal/battenshed/create.go:48`, `:74-75`).
D11's hand-back row matches only the dirty-source prefix, so the driver spawns a tackler.
That is the 2026-10-20 failure on a precondition the tackler cannot settle, and it leaves a new run-form claim behind.

### [MEDIUM] A done-marked kept pair can be removed by neither way forward

**Location:** `_lyx/discussion/decision-record.md` D2 and D7.

**Issue:** D7 leaves keeping a cancelled task from restarting to the caller, by re-scope, dependency or status.
Suppose the caller marks the kept pair's entry done.
A later `cancel --remove` runs the judge-only reconcile, which reads the entry ended and done on the fetched tip.
D2 then refuses and names `lyx cleanup --apply`.
That reconcile tears down only ended runs "that has a batten run record here" (`internal/fabricengine/doc.go:75-77`), and the first cancel removed that record.
Cleanup leaves the pair, and the cancel keeps refusing.

### [MEDIUM] A leftover task branch reads as nothing to cancel

**Location:** `_lyx/discussion/decision-record.md` D2.

**Issue:** After a cancel that kept the pair, `lyx fabric remove <slug>` removes the worktrees.
It keeps the unlanded task branch locally and on origin (`internal/fabricengine/remove.go:148-152`).
`lyx batten run <slug>` then refuses on the branch (`internal/fabricengine/remoteleftover.go:157-175`).
`cancel --remove` refuses as nothing to cancel, since D2's test reads no task branch.
No named way forward deletes the branch.

### [LOW] Marker check placement against the start owner check

**Location:** `_lyx/discussion/decision-record.md` D3.

**Issue:** `armAt` runs `refuseStartOwnedElsewhere` before `armSeed` for a start with no status file (`internal/battencli/arm.go:377-383`).
That check pulls Bolt and writes `claimPrimeOwner` (`internal/battencli/owner.go:113-118`).
D3 places the marker check only "before `armSeed`", so a start during a cancel pulls Bolt, and may write to it, before it meets the marker.
D10 places its own check before the pull and the claim.

### [LOW] End-only's step 9 actions are unstated

**Location:** `_lyx/discussion/decision-record.md` D3 step 5, D2 and D9.

**Issue:** D3 says an unreachable origin stops the cancel after step 5.
D2 and D9 have the end-only cancel post its notice, and its locks must be released, both step 9 actions.
The step list never says which of step 9's actions the end-only path runs.

### [LOW] A held batten lock is not always a raced batten

**Location:** `_lyx/discussion/decision-record.md` D4 step 2.

**Issue:** `lyx batten cleanup` try-acquires the same run lock for each slug it scans (`internal/battencli/cleanup.go:187`).
Under a concurrent cleanup the hook reads the lock busy, reruns step 1, which ends nothing, and refuses as a survivor.
The refusal names no pid, and its `ps` or Task Manager way forward finds no batten.

### [LOW] One archive tag for two differing tips

**Location:** `_lyx/discussion/decision-record.md` D5.

**Issue:** D5 keeps "the task branch's tip" under one `archive/<slug>/<tip>` tag, then deletes the branch locally and, leased to the observed tip, on origin.
When the local and origin tips have diverged, the commits unique to the side not tagged go with no archive tag.
That breaks D5's own bound.

### [LOW] Remote is not named on the removal request

**Location:** `_lyx/discussion/decision-record.md` D5.

**Issue:** `Remove` deletes the weft branch on origin only with remote set, and without it keeps an origin-only sibling (`internal/fabricengine/remove.go:144-146`, `:155-156`).
D5 names Force and Snapshot and "the ordinary origin steps", never `Remote`, so the end state it states depends on a field it leaves out.

## Focus verdicts

- **Cancel with no pair:** D2's absent path skips the reconcile and reads the owner from the pulled Bolt.
  That matches `originreconcile.go:574-576` and `fabriccli/doc.go`'s handling of a gone pair, and its refusals let the 2026-10-20 case through.
  The `--remove` form of that case is F3, and a half-gone pair is F4.
- **`--remove` snapshot and rerun:** the record names the origin-mode snapshot as a `pairteardown` change, rightly, since `RemovePair` today snapshots only with `Local` (`teardown.go:347`).
  The task branch's archive-and-delete passes the warp leftover probe, and the weft side passes through its archive tag.
  F13 and F14 are the remaining gaps.
- **Lock ownership:** each lock has a named owner, and D6's split prevents a second flock on the same file in one process.
  The status lock is held only around step 7's local removal and commit, and the push is detached, so it is never held across a network call.
  The verb's network-gate exemption covers the network calls of steps 2 and 6.
  The lock-related defects are F1, F2 and F12.
- **Declared PATTERN-pair-teardown text:** D4's order matches the declared text: owned processes first, then strands, then the loop and the run-lock wait after, then the session.
  D5 and D7 leave the claim to the cancel.
  F4 is the only place the record leaves the phase's reach unclear.
- **Interruption and offline:** the offline path names what it can no longer judge, a stale local owner and unread stopped branches, and what bounds it, end-only with nothing removed.
  An interrupted `--remove` cancel does not finish on a re-run (F3), and the end-only step list is F11.

## Rejected

- Holistic candidate: the network-gate exemption lacks a bound. D2 states what it lets through, this process's own calls, and what bounds it, the detached children keeping their gates. Not a finding.
- Holistic candidate: the warp-origin `archive/<slug>/<tip>` tag collides with existing archive-tag readers. Those readers all read the weft origin (`cleanupremotewarp.go:179-190`, `remoteleftover.go:195`, `reportengine/source.go:76`), so nothing misreads the new tag.
