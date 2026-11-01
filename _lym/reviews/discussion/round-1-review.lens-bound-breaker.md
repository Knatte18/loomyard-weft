```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker lens. I tried to break every bound, guarantee and claim about existing code in the decision record.

**Claims attacked**

- D1, "each row reads the flag when the run enters it": held (`shedengine/entry.go:30-45`, EnteredRow is persisted).
- D1, "a goto bypasses only its target's gate": held (`shedengine/goto.go:123`).
- D1, flag flipped off after `Parts-Write` or during `Parts-Plan-Write`: held. `Plan-Write`'s `!= true` check holds and its rotator archives the joined plan.
- D1, a goto onto either new row with the flag off ends in `Plan-Write`: held.
- D1, "a skip spends no bounce budget": held (`entry.go` doc).
- D1, the settled-verdict refusal names the writer "Parts-Plan-Write when Plan-Write was recorded Skipped": **broken** (F4).
- D1 and D6, "a Stuck from a failed writer or an exhausted gate halts the run as Plan-Write's does": **broken** (F1).
- D5, "a required advisor today queues no done line": held (`engine.go` watch, Required branch).
- D5, "a required advisor today notifies the parent on a held turn": held (`prompt.go:171`, QuietHold is `!isChair && !seat.Required`).
- D5, the writer keeps its pane after done: held (`prompt.go:172`).
- D5, "the chair's hold deferral counts writer seats as it counts advisors": **broken** for the post-done fix loop D9 makes routine (F3).
- D5 and D6, "a resume keeps every finished part": **broken** when the chair is lost (F2).
- D5, Accept on a writer the resume did not find "passes on them": holds only loosely, since no terminal Result exists for it (F7).
- D3, part names must be valid under `ValidateSeatName`: held for strand roles, since the grammar has no length cap (`agentname.go:86-101`). **Broken** for the directory layout (F6).
- D3, the parts check makes file order a topological order: held for parts. **Broken** for shared spots, which carry no ordering (F5).
- D7, "`--part` is a fifth mode, exclusive with the four flags": **broken** (F8).
- D8, "it writes only the plan directory; a rerun discards ..." with join-parts run once: **broken**, since nothing bounds when the verb may run (F9).
- D4 and D6, archives never nest and the preparation runs before `archiveStaleOutputs`: held (`multillm.go` Call order).
- Constraint, "no production table sets `Required` today": held (grep: only `internal/seatengine`).
- D13, reinvoke resumes through the seat probe: holds only while the chair is live (folded into F2).

**Findings**

**F1. MEDIUM, design. D5, D6 and D1: a failed writer reaches the run as an error, not a Stuck.**
A writer whose gate is exhausted or whose Accept fails fires `ender`. `join` then stops the chair (`seatengine/engine.go:361-370`).
The stopped chair has no `join.md`, so its Wait ends Died.
`mapOutcome` turns Died into an error wrapping `ErrAgentDied` (`shedadapters/multillm.go:169-181`), never into `shedengine.Stuck`.
`shedtransient.Class` marks `ErrAgentDied` transient only after a recent EAGAIN refusal (`shedtransient.go:32`), so the run persists as failed with an "agent died" reason.
That reason does not name the writer or part that failed, so the way forward is lost.
No production table has ever exercised this path, which is why the record's premise was never tested.
The D5 and D6 acceptance criteria would pin an outcome the code does not produce.
The Decision must state the mapping it builds: a Stuck carrying the failing seat and its gate findings, or an error naming them.

**F2. MEDIUM, design. D6 bound and D5: "a resume keeps every finished part" holds only while the chair is live.**
`Resume` needs a live chair (`seatengine/resume.go`, the `live.Chair == nil` refusal).
When the chair is not found, `Probe` stops every writer and returns an empty LiveTable.
`MultiLLMProducer.Call` then takes the fresh path (`multillm.go:118-131`), and D6's fresh-spawn preparation archives every part directory.
Counterexample: the tmux server dies or the machine reboots, taking the chair and every writer. The reinvoke interrupt policy (D13) then restarts every part, finished ones included.
The bound names only the lost-unfinished-writer case. It should also state the lost-chair case, or the design should keep finished parts on a fresh spawn.

**F3. MEDIUM, design. D5's hold deferral does not cover a done writer the chair asked to fix.**
`advisorWork.outstanding` skips every run that has ended (`seatengine/work.go:44`, `run.ended()`).
A writer whose run reached done has ended, even though D5 keeps its pane and D11 has it fix its part on request.
Counterexample: the chair sends a mismatch to a done writer (D9 step 2) and waits. No writer counts as outstanding, so the chair's held turn notifies the parent while the fix is in progress.
This happens on every contract mismatch, not only after a crash.
D5 states the deferral as covering writers, but nothing in it counts the post-done fix loop.

**F4. LOW, design. D1: the settled-verdict refusal can name the wrong writer.**
Counterexample: in a split run, `Plan-Write` is first recorded Skipped. Later, `lym loom goto --to Plan-Write` (D1's own way back) runs it and writes a solo plan.
History now holds both the Skipped and the later Done entry for `Plan-Write`, so "recorded Skipped" is true and the refusal names `Parts-Plan-Write`.
`appendOrFold` may fold the entries, but a Skipped entry followed by a Done one still shows both.
The rule should read `Plan-Write`'s latest history entry. Today `segmentWriter` picks by `OnDone` (`loomcli/settledguard.go:97-103`).

**F5. MEDIUM, design. D3 and D9: no check orders shared spots against parts.**
A shared spot records the parts that need it, what each needs and its exports. It records nothing it uses from parts.
Counterexample: part A, first in the file, uses a handle exported by shared spot S. S's card needs a type exported by part B, which depends on A.
No card position satisfies both, so `uses-later-target` fires on the joined plan.
D9 forbids a rejoin, so the chair's only way out is restructuring cards by hand. Failing that, the `plan` gate exhausts and the run halts.
Open risk 3 covers misplacement, not the case where no valid position exists. No Decision bounds it, and the parts check could have caught it at `Parts-Write`.

**F6. LOW, design. D3: a part name can collide with the archive directory prefix.**
The seat-name grammar `[a-z][a-z0-9-]*` (`agentname.go:86-95`) admits a part named `archive-x`.
Its directory `_lym/parts/archive-x/` reads as an earlier archive under D4's and D6's "except earlier archives" rule.
So a fresh spawn never archives that part's stale cards, and they read as this session's output.
The parts check should refuse that prefix.

**F7. LOW, design. D5: resume Accept has no Result to judge.**
`Seat.Accept` is `func(shuttleengine.Result) error` (`seatengine/table.go:89-91`).
D5 runs Accept on a writer the probe did not find, which has no terminal Result.
The Decision should state that Accept judges the outputs on disk, or how the hook's signature changes.

**F8. NIT, consistency. D7: `--part` is a sixth mode.**
`validate-plan` already documents five modes, no flag plus `--require-approved`, `--rework`, `--lens-review` and `--overlay` (`loomcli/validate.go:164-166`).

**F9. MEDIUM, design (attack surface). D8: `join-parts` is refused only by the recipe's rows and missing files.**
Nothing bounds which row may run it.
Counterexample: during Plan-Review, a `Plan-Burler-Fix` session runs `lym loom join-parts`. The recipe holds the `Parts-Plan-Write` row and the part files are still committed, so the verb succeeds.
It replaces the reviewed plan's cards, dropping the chair's shared-spot cards and the fixer's own fixes.
The same works after Webster has started.
The bound "a rerun discards whatever the chair added after the last join" names the damage but not what prevents it.
Refusing unless the run's current row is `Parts-Plan-Write` would bound it.

Tool calls:
1. `internal/shuttleengine/wait.go` OutcomeDied sites and `internal/shedtransient/shedtransient.go` Class: to confirm how a stopped chair and `ErrAgentDied` reach the run (F1). The manifest held only the call site.
2. `internal/agentname` `ValidateRole`, `SeatRole`, `ValidateSeatName`: to test the part-name claim one hop past the record.
3. `internal/agentname` `validateDashed`: the grammar and the absence of a length cap, for F6 and the role-length attack.
