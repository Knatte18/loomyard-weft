# Decision record — batten-cancel-verb

## Goal

One verb, `lyx batten cancel <slug>`, ends a batten run for good from the hub's prime, whatever its state: before or after its pair exists, live, paused, blocked, failed or awaiting.
It ends the run's processes and strands, clears the board claim, removes batten's records and scratch, and with `--remove` also removes the pair.
`lyx batten run` refuses a dirty source worktree before it writes anything, and a dirty source met later at Worktree-Create halts with that way forward and never reaches a tackler.
On Windows, where `runproc.End` signals nothing, a live run is refused until its processes are ended by hand (D4).

## Scope

In:

- `internal/battencli`: the verb, the dirty-source check on every auto-seeding verb, and the cancel notice.
- `internal/pairteardown`: a cancel phase beside `EndSession`, and a removal mode that leaves the claim to its caller.
- `internal/battenshed` and `internal/fabricengine`: a typed dirty-source error from the pair add, and Worktree-Create's precondition halt.
- `internal/shedengine` and `internal/sheddriver`: the precondition reason prefix and its hand-back row.
- Way-forward texts that tell a caller to abandon a run with `lyx board set-status` now name `lyx batten cancel <slug>`: `boardengine`'s run-lock refusal and `pairteardown`'s held-gone settle warning.
- Docs: the package docs of the packages above, the orch role stencil's run-handling line, and `docs/overview.md` where it lists batten's verbs.

Out:

- The board entry's body, scope, owner and every field but its status.
- `lyx batten cleanup` and `lyx cleanup`, which keep skipping non-done runs.
- The child run's records inside a kept pair; they leave with the pair.
- A new batten or shed state such as `aborted`: the records are removed, so nothing would read one.
- Hub-scope pause entries.
- The `ly:cli` skill, whose index `lyx help index` generates from the binary.

## Decisions

### D1. The verb is `lyx batten cancel <slug> [--remove]`, run from the prime

Rationale: it sits beside `run`, `pause`, `goto` and `cleanup` in batten's tree, and the task slug and issue name it "cancel".
The verb applies `RefuseUnlessPrime` first, so a pair agent cannot reach it.
The orch may call it: the orch is the operator's delegate, and the 2026-10-21 incident was an orch stop.
Bound: it acts on the one slug named; an omitted or reserved slug refuses, as batten's other verbs do.
Prime-only is today's `RefuseUnlessPrime` rule; `orch-any-worktree` widens it for every batten verb at once.

### D2. The run's state chooses the path; no flag chooses refuse or abort

Every non-terminal state is ended: running, paused, blocked, failed, awaiting, a seed with no status, and a run-form board claim with no records.
The verb refuses, with nothing changed, in these cases:

- The child's work has reached its parent: a Finalize past its parent-side merge (`battenshed.FinalizePastMerge`), a Finalize Done in the child's status history, or batten past Run-Shed with the child done.
  Way forward: `lyx batten run <slug>` to finish the landing, then `lyx batten cleanup --apply`.
- The entry is owned by another machine, read from the reconcile, or from the local board when origin is unreachable: way forward `lyx adopt <slug>` on this machine, or the cancel on the owner machine.
- The reconcile reads a stopped branch: the refusal names the branch's own way forward, as `lyx fabric remove` does, then the cancel again.
- The reconcile errors for any cause but an unreachable origin: way forward fixing the cause, then the cancel again.
- A live hub-scope pause entry covers the prime when the verb starts: way forward `lyx tender resume`, then the cancel again.
  A hub pause added after this check makes the verb's network calls wait at the network gate, as every prime verb's do.
- Nothing is left to cancel: no records run directory, no run-form status, no pair and no live process registered under the slug.
  Way forward: check the slug with `lyx board list`.

An unreachable origin does not refuse: the verb goes on with a Warn, as a batten resume does, and its board write and records push take the failed-push path (D7, D9).
Rationale: the operator decision of 2026-10-22 makes one verb end a run whatever its state; the refusals left protect landed work and other machines' runs.
Bound: the verb can end a working agent mid-turn and a step mid-write; it cannot end work that reached the parent or a run owned elsewhere.

### D3. Order of the cancel, each step a no-op when already done

1. The prime, slug and hub-pause checks.
2. The run-scope origin reconcile (PATTERN-origin-reconcile), then its verdict refusals (D2).
3. Read what later steps need before anything is removed: batten's status and its state and row, the child's landed reading and its refusal, and batten's in-flight tackler marker.
4. End the run (D4), which leaves batten's prime-side driver lock, run lock and status lock held by the verb.
5. With `--remove`, remove the pair (D5).
6. Remove batten's records run directory and empty its scratch beside the lock files, then commit the records side (D6).
7. Clear the board claim (D7).
8. Release batten's locks, lift the slug's own live pause entries (D8), spawn the records push and post the notice (D9).

Steps 4 to 8 run under batten's locks, so no `lyx batten run` or `lyx batten step` for the slug, nor an adopt's restart, can resume or seed the run until the records and the claim are gone.
A run that survives the end stops the verb after step 4 with nothing removed or cleared.
Rationale: every step re-reads its own state, so a cancel interrupted anywhere finishes on a re-run.

### D4. Ending the run: batten's locks, owned processes, strands, loop, session

The end runs, in order:

1. While holding the prime records side's combined write lock (`.weft/weft.write.lock`, which every committing `Fabric.Commit` takes, per `internal/fabricengine/doc.go`), and the pair's own when the pair exists, `runproc.End` for the slug, sparing the caller: batten, the loom driver, the step loop and the pair's pushes.
   The write locks are waited for at most `pairteardown`'s run-lock bound, and a spent bound refuses naming the busy lock and the cancel again as the way forward.
   They are released as soon as `End` returns, so no ended process died inside a records commit.
   While they are held, other callers that try-acquire them, such as the records push retry, the coalesced push and the origin reconcile moves, read them busy and retry by their own rules.
2. Take batten's prime-side driver lock, run lock and status lock, waited for at most the run-lock bound; a holder there at the bound is a batten started meanwhile, which one more `End` pass reaches, and a lock still held after it refuses as a survivor.
3. Shuttle's stop verb for every live strand of the pair's session, the driver strand of an `llm` child and every arbiter and producer strand among them, and for the strand named by batten's in-flight tackler marker (`StrandGUID`) in the prime's session.
4. The pair's step loop kill and loop-lock take, as the teardown's loop end does.
5. The run-lock wait, bounded by the run-lock bound, now after the end.
6. The pair's reed session end, by name when the task worktree is gone.
7. Without `--remove`, the loop's pid file mark is withdrawn and the loop lock released, so the kept pair holds no teardown mark.

Steps 3 to 7 with the pair present are a new `pairteardown` phase beside `EndSession`, so PATTERN-pair-teardown's order (owned processes, loop, session) holds.
The phase skips the quiet wait, the removal refusal probe and the pre-end run-lock wait a skipped teardown runs, since a cancel ends the holder instead of waiting for it.
A process registered under the slug still alive after `End`, or a driver lock or run lock still held, refuses with the survivors' pids and roles and the way forward `lyx tender status` and `ps` on Linux, or the `taskkill` line `pairteardown` already names on Windows.
Rationale: no Go driver, batten or `llm` driver strand is left alive when the session ends, so no drive-report writer starts, no `Run <slug> stopped` notice goes, and nothing revives a strand; that closes the 2026-10-21 incident's loop by order.
Bound: a step killed mid-write may leave the pair's working files half-written; a kept pair keeps them for inspection, and `--remove` snapshots them (D5).

### D5. `--remove` removes the pair through pairteardown, forced with a snapshot

After the end, `RemovePair` runs with Force and Snapshot, so the pair's uncommitted paths are kept under a local snapshot tag, with the ordinary origin steps: archive tag and remote branch removal.
It runs in a mode that leaves the board claim to its caller, so the claim is cleared only at step 7, under batten's locks.
A refusal `RemovePair` raises after the session end leaves the run ended with its records and claim intact; it names its own way forward, then the cancel with `--remove` again.
Without `--remove` the pair, its branches and its child records stay, with no session, no process and no teardown mark.
A later `lyx batten run <slug>` then refuses at Worktree-Create on the existing branch, so the way to rerun is `lyx fabric remove <slug>` first, which finds the pair quiet and waits for nothing; the envelope and the notice of a cancel that kept the pair say so.
Rationale: the board entry asks for the pair as an option; keeping it by default keeps the work for inspection.
Bound: `--remove` discards nothing uncommitted without a snapshot tag, and never removes a pair whose work reached the parent (D2).

### D6. Records and scratch removal reuses batten cleanup's per-slug removal

The removal of the records run directory, the scratch emptying beside lock files and the records commit are the step `lyx batten cleanup` runs for a done slug, called with the cancel's own decision instead of cleanup's done check, and under the batten locks D4 already holds.
Cleanup's own judgement stays unchanged and still skips non-done runs.
Rationale: one removal step, so the two verbs cannot drift on what a run's records are.
Bound: it removes only the named slug's run directory and scratch contents; lock files, other slugs and the pair's own records are untouched.

### D7. The board claim is cleared, and only the status

A run-form status (`boardengine.IsRunStatus`) is cleared through `boardengine` in one Bolt commit and push, in both modes, by the verb itself.
A done status, a non-run status and an absent entry are left alone, so the write is idempotent and a re-run makes it again when an earlier one failed.
A failed board push is visible through the failed-push record and creel notice (PATTERN-push-failure-visible).
After the clear the entry reads ready again; keeping a cancelled task from being started, by a hand start or by the planned automatic starts of `umbrella-daemon` and `batten-ready-overlap`, is the caller's own step: a re-scope, a dependency or a status.
Rationale: the claim is the run's board lock, so clearing it lets the entry take edits again; body, scope and owner stay the caller's.

### D8. The slug's own pause entries are lifted after the end

Each live slug-scope pause entry naming the slug is lifted by identity through `tenderengine`'s `Lift`, after the end and the removal.
With no sessions left the lift sends no resume line; a kept pair gets the push its pause deferred, and a removed pair none.
Rationale: a stale slug pause would hold a later run of the slug at its InnerRun row.
Bound: hub-scope entries and entries of other slugs are never lifted.

### D9. One cancel notice, and the records push

After the claim clear, the verb spawns the records push as cleanup does and posts one line to the orch through creel: the slug, the state and row step 3 read, and whether the pair was removed or kept, with D5's rerun step for a kept pair.
On a re-run whose records were already removed, the line says the state is no longer readable.
A failed notice only warns; the verb's JSON envelope carries the same report: what was ended, stopped, removed, cleared and lifted, and any snapshot tag.
The notice goes even when the orch itself ran the verb.

### D10. Every auto-seeding batten verb refuses a dirty source before it writes anything

On a start (no status file) and on a resume whose status is still at Worktree-Create with no pair, the pre-run of every auto-seeding verb (`battenAutoSeedVerbs`: `run` and `step`) reads the source worktree's tracked uncommitted paths and refuses when there are any.
The source worktree is the one fabric's pair add forks from, which is the prime today.
The probe is the one the add uses: `internal/fabricengine/dirtiness.go` reads tracked paths only (`--untracked-files=no`), and `trackedDirtyPaths` returns them verbatim.
The refusal lists the paths and the way forward: commit or revert them in the source worktree, then the same verb again.
It runs before the seed, the board pull and the claim, and in the caller of any path that opens a window, before the window opens, so the orch reads the refusal from its own call.
A resume past Worktree-Create runs no such check.
Rationale: the operator decision of 2026-10-20; batten owns the precondition, so the orch need not check it.
Bound: untracked files do not refuse, matching the add.

### D11. Worktree-Create halts on a dirty source with a precondition reason, never a tackler

Fabric's pair add returns a typed dirty-source error carrying the tracked dirty paths instead of a bare message.
Worktree-Create maps it to a Halt Stuck whose reason starts with a new shed constant, the precondition prefix, and names the paths and D10's way forward.
The Go driver's rule table gains a row beside the bounce-budget-exhausted row: a blocked stop whose reason, or status error, starts with the precondition prefix hands back, ahead of every hand-over, arbiter and tackler row.
Rationale: the source can turn dirty between D10's check and the add, and today a blocked stop no other row matches goes to the tackler (`internal/sheddriver/doc.go`), as on 2026-10-20.
Bound: the row only hands back and never resumes; only a producer that sets the prefix reaches it, and Worktree-Create sets it only for the typed dirty-source error.

## Constraints

- PATTERN-no-denied-recovery: the dirty-source way forward says "commit or revert" and names no `git stash`.
- PATTERN-refusal-way-forward binds the new shed constant, the driver row and Worktree-Create's reason through `internal/shed*`; batten's own refusals name a way forward too, as its existing ones do.
- PATTERN-shed-run-directory: the run directory, scratch, lock and tackler-marker paths come from `shedrun`.
- PATTERN-sole-parsers: the precondition prefix is read through the same stop and status reading the budget-exhausted row uses.
- The `--window` window is created with `remain-on-exit off` (`internal/reedengine/doc.go`), so it closes when the ended batten exits and needs no step of its own.

## Auto-mode assumptions

- D1: the name `cancel`, with no `abort` or `abandon` alias.
- D2: no confirmation flag for a live run; an unreachable origin goes on with a Warn.
- D5: the pair kept by default.
- D9: the notice posted when the orch is the caller.

## Open risks

- A raw git child of an ended process may leave an `index.lock` in a records repository if it ran outside the combined write lock; the next commit there then fails and names the lock.
- Processes an older binary started are not registered; the driver-lock and run-lock re-check catches the driver, and other such processes stay for `lyx tender reap`.

## Acceptance criteria

- A blocked run with no pair, as on 2026-10-20: the cancel stops its tackler strand, removes its records run directory and scratch contents, commits and pushes the records side, clears the board status, and the runs overview no longer lists it.
- A live run with a Go child mid-step and an agent turn running: batten, the driver, the loop and the pair's session end; no drive report is written, no stop notice is posted, one cancel notice is posted; the pair stays with no session and no loop mark; the claim is cleared and the records are removed.
- A live run with an `llm` child driver: the driver strand is stopped before the loop end, and no drive report or stop notice follows.
- `--remove` on a live run with uncommitted pair paths: the pair is removed with a snapshot tag holding them, and the claim ends cleared although batten's record read `running`.
- A paused, an awaiting and a failed run each end the same way.
- A `lyx batten run <slug>` started during a cancel neither resumes nor seeds the run before the cancel ends.
- Each D2 refusal is reached with nothing ended, removed or cleared, and its message names its way forward; an unreachable origin cancels with a Warn.
- A survivor after the end refuses with its pid and role, and nothing is removed or cleared.
- A cancel interrupted after any step completes on a re-run, the claim clear included; after a full cancel with `--remove`, a re-run refuses as nothing to cancel.
- After a cancel with `--remove`, `lyx batten run <slug>` starts fresh; after one without, it does so once `lyx fabric remove <slug>` has run.
- The slug's own pause entry is lifted and a hub entry is left.
- `lyx batten run` and `lyx batten step` on a dirty source, `run` plain and in a window, refuse naming the paths: no seed, no claim, no window, no tackler; a resume at Worktree-Create with no pair refuses the same way; untracked files alone do not refuse.
- A source dirtied after the start check halts Worktree-Create with the precondition reason, and the driver hands back with no tackler spawn.
- `lyx batten cleanup` still skips non-done runs.

## PATTERN changes

- `PATTERN-pair-teardown` — changes: Every teardown of a pair goes through `internal/pairteardown`, which ends the pair's owned processes and then its reed session before any worktree is removed; its driver checks are skipped for a pair that is paused, ended, owned by another machine, or done locally with batten no longer running, never its run-lock wait while the task worktree is present, which then runs before the owned processes are ended; a batten cancel's phase instead ends the owned processes and strands first and waits for the run lock after, and its removal leaves the board claim to the cancel, which clears it.

## Notes for the plan writer

- Cleanup's per-slug step is `cleanupSlug`; split its removal from its done judgement rather than copying it.
- The precondition prefix belongs beside `shedengine.ReasonBounceBudgetExhausted`, and its rule row beside `budgetExhausted` in the driver rules.
- The batten tackler's in-flight marker under `shedrun.TacklerDir` carries the strand to stop.
