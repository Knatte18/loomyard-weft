```yaml
lens: walkthrough
outcome: reported
```

Lens: walkthrough. I ran each decision's mechanism end to end on one concrete case.

**Findings**

1. **BLOCKING — D3, which runs are held.** The record says "owned by another machine before this call" but does not say which Bolt that owner is read from.
   - Case: Neve owned beam X. OSL-1046 later adopted X and pushed that owner to origin. Neve's local Bolt is stale and still names Neve.
   - Walking it: adopt on Neve reads its owners before the reconcile, sees X as owned here, and writes no hold. After the beam steps, dobby finds the run uncovered and gives it a full pass.
   - That is the incident the task exists to prevent.
   - Fix: read the owner from the Bolt the step-1 reconcile left, before the owner write. That matches `PATTERN-owner-machine` ("a start's read from the Bolt it just pulled").

2. **MEDIUM — D3 bound and Open risks: a covered run still gets continue passes.** The record claims dobby gives a covered run no full pass. That is true but incomplete.
   - In `planPass`, when the run is covered and not on a lift, it returns `passContinue` if the run is at Run-Shed and no full pass is owed. The incident run, a loom child at Code-Bouncer, is at Run-Shed.
   - Walking it: a fresh dobby on the new machine has no state for the run, so `FullPassOwed` is false. It spawns a continue pass under the hold.
   - That pass is refused, because no dobby state names it as driven. Per `PATTERN-owner-machine`, the refusal then reads as a full pass owed, which ends the passes. So no run happens, but a pass process is spawned.
   - A machine that drove the run in an earlier ownership episode (A→B→A) may still hold a dobby state naming it. Then the continue pass is not refused, and its work waits only at the pause gate.
   - The record should state this path and its bound, not only the full-pass rule.

3. **MEDIUM — D5 `landed`.** `RunRecord` reads `shedrun.StatusFile(prime, slug)`, which is under the prime's records. `movedRuns` drops entries whose board entry is `Done`, and drops runs whose record is missing or done.
   - Case: a run that landed on the other machine. The reconcile fast-forwards the prime's records and Bolt. If the record is removed when the run ends, or the board entry turns done first, nothing "reads done after it" is ever observed.
   - Defining landed as "in the before list, absent from the after list" also counts runs that left for other reasons, such as a cancel or a moved entry.
   - Define `landed` from the board entry reading done after the reconcile.

4. **MEDIUM — D2's claim about stops.** The record says the warp rewrite and diverged cases are stops that adopt already answers through its local-commit stop.
   - Per `internal/adoptcli/doc.go`, adopt refuses only on a stop of Bolt or of the prime's records branch. Its per-slug branch stops apply only to adopted beams or moved runs read as `owned_elsewhere`.
   - Case: the prime's code branch was rewritten on origin. The reconcile reports it `rewritten` and leaves it in place, and adopt goes on with a stale prime warp, naming nothing.
   - The claim is false for the prime's code side. Either adopt should report that stop with `lym fabric pull` as the way forward, or the record should state the gap.

5. **LOW — D3/D4, Finalize-halted child.** The bound says `lym tender resume &lt;slug&gt;` also lifts a hold. That lift gives the run a dobby full pass, but it never runs D4's `lym loom resume` for a child halted at Finalize's owner check, so that child stays halted.
   - The record should say this, or name `lym beam go` as the only lift that resumes such a child.

**Per decision**

- **D1** — `lym adopt X` with X owned elsewhere. Purpose met, given D3 is fixed.
- **D2** — prime code branch rewritten on origin. Not met (finding 4).
- **D3** — stale local owner, and a run at Run-Shed. Not met for the stale owner (finding 1); covered but with a continue-pass gap (finding 2).
- **D4** — `lym beam go X` with one held run and one Finalize-halted child. Met: the lift triggers dobby's full pass and `StartFirstPass` races it under the driver lock, which allows one.
- **D5** — a run that landed elsewhere. At risk (finding 3).
- **D6** — a released task with an empty status. Met.

**Tool calls**

- One Bash call read three things my context lacked:
  - the covered branch of `planPass` in `internal/dobbyengine/passes.go` (I had seen only its doc line), to check which passes a covered run still gets;
  - the `RunRecord` and `ChildStatus` seams in `internal/adoptcli/deps.go`, to learn where the run record is read from;
  - `movedRuns` in `internal/adoptcli/adopt.go`, to check what before and after lists D5's `landed` would compare.
