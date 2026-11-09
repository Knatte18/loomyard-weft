```yaml
lens: bound-breaker
outcome: reported
```

Restating my task: bound-breaker lens. I tried to break each bound, guarantee and claim about existing code in `decision-record.md`.

**Claims attacked**
- D1 bound "the worker takes its request only from argv that the caller's spawn and the D2 resume build": **broken**. Anyone can run the hidden verb.
- D1 "the caller knows its worker by pid; it follows another pid's record when the subjects match": **broken**. The caller can follow a stale record.
- D2 "the winner writes the record, `running` at `reconcile`, before any step runs", read together with resume: **broken**. A resuming winner would wipe the steps already done.
- D2 "`--discard` refuses while a worker is alive": **broken**. Nothing stops a race with a worker that is starting.
- D2 rationale "every step is safe to run again, so a resume never repeats a step's effect", and the bound "a resume does not re-read origin for the steps before its first step not done": **broken**. Nothing limits how stale a resume can be.
- D2 "the resume keeps the record's flags and adds the call's": **broken**. The merge can produce `--keep-local` plus `--drop-local`.
- D4 "the accessor reads without a lock; an unreadable record counts as no record": **held**. `internal/state` replaces the file atomically, so a reader sees a whole file or none.
- D4 "the orch watcher and dobby of the subject's worktree" covers the window: **broken** for `root`. Beams with no owner of their own are not covered.
- D4 "the record names the subject" for an `--all` record: **broken**. The record holds the flag, not the set.
- D4 bound "covers only until the subject's owner write": **held**.
- D5 "the adopt check, the dobby's owner lift and a hibernate wake never lift it": **held**. `liftOwnerPause` lifts only `OwnerCaller`, and the wake's `holdRuns` adds no entry where one stands.
- D5 bound "an older check across the upgrade can lift a legacy hold once": **broken**. It understates what the old green end does.
- D5 "`lym tender pause` answers `already_paused: true`": **held** (`tenderengine/pause.go:115-118`).
- D6 "no run goes on under the old lym": **held** for slugs D5 pauses. Slugs that D5 skips because they hold another caller's entry go on only when that entry is lifted; I found no case without a lift.
- D7 bound "the dobby starts no adopted work": **held**, with the same caveat as D6.
- D10 bound "a stale Bolt skips the adopt, then `beam start` refuses naming `lym adopt`": **broken**.
- D11 "pause first, so no pass starts while it archives": **broken**, though only in a minor way.
- D11 claim "a tag push moves no branch that `PATTERN-owner-machine` reserves for the owner": **unsupported** by the PATTERN text.
- D11 claim "the remote-warp cleanup reads the tags": **partly**. The readers I found are the report engines, which would also read a `root` slug.
- D12 "lists the tags whose commit the slug's origin branch does not contain": **broken**. It lists noise from slugs that were torn down.
- D3 "`check` through today's start, which skips a check already running": **held** (`check.go:524,564` answers `already_running`).
- Constraint "a re-run writes the same owners": **broken** when the re-run is a stale resume, the same counterexample as the staleness finding.

**Findings**

1. MEDIUM, design. `decision-record.md` D10 bound (line 183). The bound says that when `--here` skips the adopt on a stale Bolt, `beam start` then refuses naming `lym adopt`. A stale Bolt only arises when origin is unreachable. In that case `BeamUp`'s prepare reconcile also cannot reach origin; it only warns and goes on, so `beam start` starts the orch and dobby on a subject another machine owns. Counterexample: origin is down and the local Bolt still names this machine, so `lym s` ends `owned_here` and `beam start` brings everything up. Evidence: `internal/orchcli/start.go:446-447` (unreachable only adds a warning). The bound has to state what it lets through.

2. MEDIUM, design. D2 resume (lines 55-57 and 67-68) against the Constraint at line 221. A resume skips `reconcile` and reuses the moved runs, pause set and stop verdicts stored in the record, however old they are. Then `owner` runs `SetOwners` with `Takeover: true`, which writes over any machine's owner.
   - Counterexample: a worker is killed between `pause` and `owner`. Days later a third machine has adopted the subject. `lym s` reads the subject as foreign-owned and resumes at `owner`.
   - That resume skips the local-commit stops and the pairs added since, pauses nothing released since, and takes the owner from the third machine.
   - Nothing bounds a record's age or makes a resume after `reconcile` re-reconcile.
   - Evidence: `internal/boardengine/ownerset.go:43-44,179,195` (Takeover writes over a foreign owner) and `adopt.go:256-262` (today the reconcile always runs first).

3. MEDIUM, design. D2 `--discard` (line 61). The record does not say that `--discard` takes `adopt.lock`.
   - Interleaving: `--discard` reads the worker as dead. A concurrent `lym adopt` then spawns a worker, which takes the lock and writes a `running` record. `--discard` then deletes that fresh record.
   - The live worker runs on with no record. D4 then reads no record and stops suppressing, so the orch watcher stops the orch mid-adopt, which is the original failure.
   - The focus asked about exactly this race, and D2 leaves it unbounded.

4. MEDIUM, design. D4 subject coverage (lines 92-96). The record covers only "the orch watcher and dobby of its worktree", meaning the prime for `root`. A beam project with no owner of its own reads the prime's owner through `ParentOf` (`PATTERN-owner-machine`, "a beam project with no owner recorded" reads its parent's).
   - During `lym adopt root`, the reconcile makes such a beam read as foreign-owned. That beam's own watcher and dobby are not under adopt, so they stop the orch and exit.
   - Its runs are driven by the beam's dobby, which D4 does not keep idle.
   - Evidence: `internal/dobbyengine/loop.go:494` (exits on the beam's effective owner) and `internal/orchengine/owner.go:30-44`.

5. MEDIUM, design. D1 bound (line 39). The bound says the request comes only from argv built by the caller's spawn and the resume. `lym adopt worker` is a hidden cobra verb that any shell or agent can call directly.
   - A direct call skips the caller's validation (`namedEntries`, the keep/drop conflict, the `--here` routing).
   - It also skips the caller's resume/refuse decision in D2. Only the lock stops a second worker; an unfinished record with other subjects does not.
   - The bound states neither what a direct call skips nor what limits it. Evidence: `adopt.go:208-233` is where the validation lives today.

6. MEDIUM, consistency. D11 rationale (line 204). It claims "a tag push moves no branch `PATTERN-owner-machine` reserves for the owner". The rule text bars a run's "push" on a machine that reads its entry as owned elsewhere, not only a branch push. D11 pushes archive tags of run tips from the old owner precisely after it reads the entry owned elsewhere. The record neither declares a PATTERN change nor cites rule text that exempts tag pushes. Evidence: `pattern/PATTERN-owner-machine.md:7,9`.

7. LOW, design. D1 follow (lines 28-29). A caller whose worker lost the lock reads "the record". Before the winner has written, that record can be the previous `done` record for the same subjects, with another pid. The caller then follows it and prints a stale success envelope, exit 0, while the real adopt is still running. The pid check proves only that the record is not this caller's, not that it is current.

8. LOW, consistency. D2, line 49 against lines 55-57. "The winner writes the record, `running` at `reconcile`, before any step runs" also covers a resuming worker. Taken literally, that write replaces the done steps and the stored inputs that the resume needs.

9. LOW, design. D2 flag merge (line 56). A record made with `--keep-local` that ends `stopped`, re-run with `--drop-local`, merges to both flags. `adopt` refuses that pair (`adopt.go:213-215`), leaving `--discard` as the only way on.

10. LOW, consistency. D4 with D2 (lines 44 and 92). The record stores the request, including `--all`, rather than a resolved subject set ("`--all` matches a record made by `--all`, whatever set either resolves to"). D4's test "the record names the subject" is undefined for an `--all` record until the accessor resolves `--all` against the board, and the record does not say it does.

11. LOW, design. D5 bound (line 133). It says an older check running across the upgrade "can lift a legacy hold once". The old green end also does three more things:
    - It calls `BeamUp`, which starts the orch despite `--no-orch`.
    - It starts first passes of the runs it lifts.
    - It runs `lym loom resume` on children halted at Finalize's owner check whether or not any hold was lifted.

    Evidence: `internal/adoptcli/green.go:73-80,157-187`.

12. LOW, design. D11 step 1 (line 192). "So no pass starts while it archives" fails when the owner pause fails. `ownedElsewhere` records `OwnerPauseFailed` and gives the run a full pass on the next probe tick while the archive retries. Evidence: `internal/dobbyengine/owned.go:39-43` and `passes.go:124-127`.

13. LOW, design. D12 (line 212). Teardown and `--drop-local` archives belong to slugs whose origin weft branch the teardown deleted. "Whose commit the slug's origin branch does not contain" is then true for every such tag, so `--archives` lists every old teardown archive under the subject alongside the old owner's tips.

14. LOW, consistency. D11 names `root` as the slug of the prime's archive tags (line 196). The report engines read every archive tag's slug through `ArchiveTagSlug` and would report `root` as if it were a run. Evidence: `internal/reportengine/runs.go:139`, `cost.go:136`, `timing.go:242`, `raddle.go:71`.

**Tool calls**
- Read `orchcli/start.go`'s handling of an unreachable origin and green.go's `resumeHaltedChildren`: the context lacked BeamUp's unreachable path and the body of that function.
- Grepped for `ArchiveTagSlug` readers, `SetOwners`' `Takeover` semantics and the check start's already-running skip: the context lacked the callers one hop past D11/D12 and the owner setter's contract.
