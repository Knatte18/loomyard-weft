```yaml
lens: bound-breaker
outcome: reported
```

**Bound-breaker lens.** I attacked every bound, guard, ordering promise and code claim in the decision record.

Claims attacked:

- D1:37–38, "every `EnsureLive` caller starts only the beams it names", and the landing-hold exclusion. Held. The callers are `adoptcli/deps.go:122`, `fabriccli/anchordeps.go:265,272`, `beamcli/cli.go:160` and `beamcli/deps.go:82`, plus `orchcli/start.go:271`, which goes away. Each is a site the record names.
- D4:69, "`runproc` has no role-scoped end". Held: `runproc/end.go` exports only `End`, `EndSparing` and `EndProcess`.
- D4:70, a hand `lyx batten run` restarts the dobby unless the hold stands. Held, through `ensureParentDobby` (`battencli/cli.go:506`).
- D5:86, the drop on load "the way `migrate.go` drops `tier` and `type`". Held: `migrate.go:22-23,43-44`.
- D5:91, the mixed-version bound: an old build "still reads and writes task owners". **Broken** (F2).
- D9:141–142, the accept ordering (stop the dobby, write the hold, re-check) together with the member's Worktree-Create re-read. Held for members on this machine: either side sees the other, and the re-check also catches any open member entry.
- D9:144, "restarts the beam's dobby only when this accept stopped it", taken across a crash between accept steps. **Broken** (F1).
- D9:145, a failed status write keeps the request. Held: the beam stays held and members are refused until a write succeeds.
- D9:148, recovery after a crash between removing the request and Done. Held.
- D9:157, "a status left on the wrong side of the hold stands at most until that wake". Held, because every hold crossing is a Beam-Await write.
- D9:158, the slot rule. Held as a claim. The mechanism needs the board status, because dobby's `RunView.Producer` holds batten's own producer (`dobbyengine/passes.go:29-31`). D9:153 already says every reader reads the board.
- D6:104 together with D11:183, the adopt-order cases (root then G, and G then root). Both held. The case of adopting root alone leaves an undriven window with no stated bound (F4).
- D10:165–167, "`--anchor` … so a resume keeps it" and the re-run bound. Partly broken (F5).
- D11:184, cancel refuses while `BeamLandRefusal` refuses. Partly broken: it reads a stale local Bolt (F3).
- D12:193, the restarted dobby sends no second notice. Held.

Findings:

- **F1 — MEDIUM, design, D9 (`decision-record.md:141,144,148`).** A crash during an accept can leave the beam at Beam-Await with no dobby, and nothing names `lyx beam start`.
  - Counterexample: accept 1 stops the beam's dobby, then the driver dies before or after writing Parent-Align. The respawned driver re-enters Beam-Await: its entry write lifts the hold (D9:140), the request still stands, and it accepts again. This time the dobby is already dead, so "this accept" did not stop it.
  - If the re-check now fails (a member was opened), it writes Beam-Await, does not restart the dobby, removes the request and keeps waiting.
  - The members are then undriven, and the posted failure does not name `lyx beam start <slug>`. The interrupted-landing notice (D9:148) covers only the case with no request standing.
  - Fix: the record has to say how an accept knows the dobby it finds dead was stopped by an earlier accept (for example a marker), or have the failure notice name `lyx beam start`.
- **F2 — MEDIUM, design, D5 and D10 (`decision-record.md:87,91,164,172`).** D5's mixed-version bound is false, because today's build refuses a project that carries a recipe.
  - Evidence: `boardengine/task.go:288-289` calls `checkProjectFields` for every project. That function refuses `{"recipe", t.Recipe != ""}` (`task.go:307`): "a project carries no recipe".
  - It runs in the same normalising pass that fixes up `Created`/`Revised` and empties `RunSettings` (`task.go:270-279`). That pass looks like the load path, though I did not confirm the caller.
  - So once `lyx beam create` writes `recipe: pawl` on a beam project, a machine on the old build refuses that entry. It does not "still read and write task owners".
  - D10:172 already says "a beam project with no recipe reads as pawl", so writing the recipe is not needed. Fix: either stop writing `recipe: pawl`, or restate the bound as "an old build cannot use the board until it is updated".
- **F3 — LOW, design, D11 (`decision-record.md:184`).** The cancel refusal reads this machine's local Bolt, and the record states no bound on how fresh it is.
  - Counterexample: a member entry is upserted with `project: G` on machine B, where G reads Beam-Await and is not held, so it is accepted and pushed. Machine A, whose Bolt is not yet pulled, runs `lyx batten cancel G --remove`. `BeamLandRefusal` passes and the pawl run and pair go.
  - The member is left naming a beam that has no pair. The way back is `beamForkPrecondition`'s create or adopt way forward, but the record claims "it would orphan the members" (support log Q13) and leaves this path open.
  - Fix: run an origin reconcile before the check, or state the bound.
- **F4 — LOW, design, D6 (`decision-record.md:100-105`).** Adopting root alone leaves G's pawl run with no driver on either machine, and the record states no bound for that.
  - Setup: G is owned by A, and B runs `lyx adopt root`. A's prime dobby exits on its owner read, so G's pawl run on A loses its parent dobby. A pass on A still reads G as its own (D11:183), but once its driver dies the run has none.
  - A hand pass on A goes through `ensureParentDobby`, which refuses because the parent is owned elsewhere ("the run stays undriven", `battencli/cli.go:518-522`). `lyx beam start G` on A resumes no run (D1). B reads the run as owned elsewhere and does not resume it (D6:105).
  - The landing is blocked either way (D10's parent-owner refusal names `lyx adopt root`), but the record should state this window and what `lyx batten status` shows during it.
- **F5 — LOW, design, D10 (`decision-record.md:165-167`).** `--anchor` is kept only once the first pass has written batten's run record.
  - Counterexample: the first pass dies before writing that record, which leaves the "flagged beam with no run status" case. A re-run of `lyx beam create <slug>` without `--anchor` "gets its first pass again" and forks with the default anchor, silently.
  - The bound names repeating `--anchor` only for the covering-pause case. Fix: record the anchor where a re-run reads it (such as the entry or a create marker), or extend the bound to the died-pass case.

Tool calls:

1. Grepped `EnsureLive` and `SpawnDetached` callers and listed `runproc/end.go` functions, to test D1:37 and D4:69. My context had neither the caller list nor `runproc`.
2. Grepped `migrate.go` for tier/type and the board's recipe validation, to test D5:86 and the old-build tolerance of `recipe: pawl`. Not in my context.
3. Grepped `battenshed` for the quiet/lock-wait notice and listed `dobbyengine` files, to test D9:151 and D7:112. Inconclusive, and not used.
4. Read `boardengine/task.go:296-325` and the caller of `checkProjectFields`, to confirm that a project's recipe is refused. Not in my context.
5. Grepped `innerrun.go` for quiet and driver-dead lines, to test D9:151. Inconclusive.
6. Read `boardengine/task.go:270-292`, to see which pass runs `checkProjectFields`. Not in my context.
