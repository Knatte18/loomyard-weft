# A pause holds every prompt, verify command and new batch in its scope

A `lyx tender pause` now holds everything lyx would start in its scope, so the operator no longer kills processes or re-pauses by hand.
It closes four gaps: a gate re-prompt reaching a covered session, a verify command running on under a pause, `lyx creel interrupt` refused under a pause, and webster's Master opening a batch under a pause.

Closes #762

## What changes

- **Shuttle refuses to type into a covered session.**
  `sendVerified`, the one typing site for every shuttle line but `Runner.SendConditioned`'s, reads the target session's own pause coverage after the idle wait and just before typing, and refuses with a new `ErrPaused`.
  The slug comes from the target strand's name as reed reports it, so a hub-level runner sending into another pair is judged by the target, not by itself.
  Keys (`Interrupt`, `InterruptRestoring`, `Inject`, the pause's own Esc) stay outside the check.
- **Callers hold or report `ErrPaused`.**
  - The gate re-prompt and pending send are held like `ErrSessionBusy`: no attempt is spent, and the boundary is re-evaluated once the run reads uncovered.
  - Run start and the deferred task prompt wait through `pauserecord.WaitUncovered` on the slug the refusal carries, then send again.
  - Seat advisor notices retry with a doubling interval up to five seconds instead of being dropped.
  - Creel delivery maps `ErrPaused` to a held outcome, so the inject stays queued until the resume.
  - `lyx tender resume`'s resume line claims the session into the entry that refused it.
  - `lyx loom resume` and `lyx shuttle send` return the refusal with its way forward.
- **Shuttle's gate loop holds its boundary while covered.**
  A gated Done arrival under cover runs no gate closure until the run reads uncovered, then evaluates as if the arrival had come then.
  Off-boundary `Expired` probes keep running, since they send nothing.
- **`verifytree.Verify` holds its command under a pause.**
  `verifytree.Site` takes an optional hold, told by the caller as two functions: one reading whether a covering pause has passed its grace end, one waiting until no pause covers the scope.
  A Verify entered under a covering pause starts nothing until the resume.
  A running command is cancelled once the covering entry passes its grace end; Verify marks itself `paused`, waits, and starts over from its dirty-tree check.
  A pause resumed inside its grace lets a nearly done verify finish and record its result.
  The loomshed and webster verify gates, landingshed's Merge-In, Finalize and Publish, and `lyx webster verify` all tell their scope's hold.
  A site whose run holds a parent lock reads hub entries only.
- **`verifyrun.Run` cancels SIGTERM-first.**
  A new cancel in `internal/proc` sends SIGTERM to the command's process group, then SIGKILL after the wait delay through its own timer, and `Run` returns only once the group is empty or that SIGKILL is sent.
  SIGTERM matters because `lyx gate test`'s terminating-signal handler kills its tied `go test` tree and frees its slot; a bare SIGKILL leaves the test binaries running.
  It applies to every `verifyrun.Run` caller, including batten's `deploy` and `Command` rows and the card `**Verify:**` rerun on their timeouts.
  `proc.ConfigureGroupKill` keeps its immediate group SIGKILL for `gitexec` and the `lyxbin` test kit.
- **`lyx creel interrupt` interrupts a covered strand.**
  Creel claims the strand into the covering entry, re-reads under the receiver's pane lock that the entry is still live, then sends the guarded interrupt; `lyx tender resume` then gives the strand its resume line.
  The privilege rule, another sender's hold and an unreadable record still refuse.
  Creel's delivery worker records nothing in the entry.
- **`lyx webster begin-batch` and `recover-batch` refuse under a pause.**
  They read coverage before any state change, claim Master's session into the covering entry, and refuse with a message telling Master to end its turn and rerun the verb after the resume line.
  A failed GUID lookup refuses with its own text and records nothing.
  An unreadable record refuses nothing and logs one Warn.
  `record-batch`, `await-batch` and `status` stay open.
- **`pauserecord.ClaimSession`** records one session in the live entry covering a slug, under the record lock; it records nothing for an uncovered slug or a session another live entry owns.
- `PATTERN-pause-holds-prompts` and `PATTERN-verified-tree` carry the new invariants, and the module docs are updated.

## Design decisions

- The check sits at the one typing site and is keyed on the target, so it holds for every caller.
- The orch runner is told no pause geometry, so `lyx orch start` keeps starting under a hub pause and its start lines read uncovered.
- Tender and creel runners now get the hub's pause geometry through `SetPause`.
- A runner told no geometry, and an unreadable record, read uncovered with one Warn per change of error, so a corrupt file never freezes sends.
- Verify stops a command only after the covering entry's grace end, matching the pause's own grace, and never records a stop as passed or failed.
- Out of scope: driver chains, parked pushes, background shells, Monitors and webster forks under a pause (`run-process-ownership`), and what `lyx tender pause` itself interrupts.

## To check by hand

- Windows: `verifyrun`'s cancel ends the shell alone, so the test tree keeps running there; the `verify-watchdog` board note owns that gap.
- An `Expired` or liveness finalize of a gated run under cover reaches a held Verify and waits there until the resume.
- A tender resume landing between creel's re-read and its key, whose line then fails busy, leaves the interrupted strand with no resume line.
- A pause landing after the last coverage read, during an input-box engine's deferred submit, lets that already-typed line be submitted.
- After `begin-batch`'s failed GUID lookup no line wakes Master; it stays idle until the operator prompts it.
- A session claimed by creel interrupt or the batch verbs that is busy again at the resume gets one extra carry-on line.
- In `internal/shuttleengine`, `boundaryHeld` must be cleared wherever the boundary is cleared or re-arrived, including a turn change; a late fix touched this, so check that no path leaves it set.
- The `PATTERN.md` overview trims: check that every cut is one the task intended and that none is left undeclared.
- The verify gate failed once on `cmd/lyx.TestCLIForm_EveryCommand` because the new batch verbs' `Long` text exceeded the CLI form cap; two fix commits shortened it (`0a9f5279b`, `3a005438a`).
  Check the shortened help text still reads clearly.
