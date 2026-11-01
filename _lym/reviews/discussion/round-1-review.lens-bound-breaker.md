```yaml
lens: bound-breaker
outcome: reported
```

Lens: bound-breaker. I attacked every claim and bound the record states.

**Claims attacked**
- D4: dobby's start already waits on a pause covering the slug — held (`internal/dobbyengine/starts.go` `readyToStart`/`heldByEntry`).
- D4: holds are written before the owner write, and an owner entry turns into the hold in one `pauserecord.Update` — held.
- D4 bound: a push child under `OwnerEntryExempt` loses its exemption under the hold — held (`internal/pauserecord/hold.go:293` exempts only `OwnerCaller`).
- D4/D6: the fix task filed after a red check starts while the moved runs stay held — **broken** (F1).
- D4/D6: a re-run of `lym adopt` while the fix task stands released and not started — **broken** (F2).
- D5/D8: the green end cancels a halted fix task, and the front start hold lifts once the task is "done or removed" — **broken** (F3).
- D6 bound: at most `adopt_fix_tries` per subject, at most one at a time, slug `adopt-fix-<subject>` cut to the slug limit — **broken** (F4).
- D6/D8: lym files a task at runtime through `boardengine` — the bound against the run-agent task guard is unstated (F5).
- D7: the closed state set tells finished, should-run and should-not apart — **broken** for a halted fix task (F6).
- D8: `front` and `on_land` are written only through `fronttask`, and only `lym board upsert`'s refusal is named — partly open (F7).
- D8 bound: the closed action table stops a board edit from making dobby run a command — held.
- D5: a start refused for a held driver lock is no error — held. The bound on a second adopt during a running check is stated — held.
- D6: the darn default keeps the perch on (`light`, `code_max_bounces` 1) — held (`lym board profiles`: darn default `light`).
- `PATTERN-tolerant-records` for `front` and `on_land` — held (`internal/boardengine/doc.go:146` keeps unknown keys on every write).
- D2's claims about `ReconcileOrigin` — not attacked; the reconcile code is not in my context.

**Findings**

F1 — BLOCKING, design, D4 with D6 and D8: held runs fill `max_runs`, so the fix task never starts.
- `slotsTaken` counts every listed run not marked owned elsewhere, and a pause-covered run is not exempt. `startReady` breaks once `running >= cfg.MaxRuns`.
- Counterexample: adopt a beam with 3 or more moved runs (default `max_runs: 3`). All are held, the check is red, and `adopt-fix-<beam>` is filed released. Dobby's start step never reaches it, so the fix never lands, the check never reruns and the holds never lift.
- The record never says the front task bypasses the slot limit, or that held runs take no slot.
- Evidence: `internal/dobbyengine/starts.go:62-80` (`startReady` break), `:158-167` (`slotsTaken`); `internal/dobbyengine/template.yaml:1`.

F2 — BLOCKING, design, D4 second bullet with D6: a re-run of adopt holds the fix task itself.
- D4 holds "each task under an adopted subject that is released with an empty status", with no owner condition and no exclusion for a front task.
- Counterexample: the fix task is filed, released and not yet started (dobby down, or F1). The operator re-runs `lym adopt <subject>`, as D6's notices tell them to. The fix task gets a `HoldCaller` entry, and the new check runs on unchanged code and goes red. D6's "an open fix task is never filed twice" files nothing.
- Deadlock: only a hand `lym tender resume` frees it, and D7 reports the fix task as `waiting_for_check`.
- Evidence: decision-record D4 bullets, D6 "never filed twice"; `internal/dobbyengine/starts.go:53` `readyToStart` (covered ⇒ no start).

F3 — BLOCKING, design, D5 green end with D8 and D6: cancelling the halted fix task leaves a draft `front` entry.
- `lym batten cancel` clears the claim by writing `draft` (`clearRunClaim`). The entry is neither done nor removed.
- D8 lifts the start hold only once the front task "is done or removed". So after a green check, every new start in that worktree stays held for good.
- The draft entry is still "open", so D6 never files a fix again on a later red check.
- Without `--remove`, the slug cannot run again either.
- The record rests on an unchecked fact about what cancel leaves.
- Evidence: `internal/battencli/cancelcmd.go:491-503` (`clearRunClaim` writes `StatusDraft`), and the cancel help text ("--remove must run before the slug runs again").

F4 — MEDIUM, design, D6 slug rule: the cut can merge two subjects into one slug.
- `MaxSlugLength` is 32 and `adopt-fix-` is 10 characters, so the subject is cut to 22.
- Counterexample: beams `payments-reconciliation-a` and `payments-reconciliation-b` share one fix slug. One subject's red check sees the other's open fix and files none. One subject's green end cancels the other's halted fix (D5).
- An existing unrelated entry with that slug would take the upsert as an edit of itself.
- This breaks D6's "one at a time per subject" bound and the bound on D5's cancel. The record cites the limit only as "the board's slug limit".
- Evidence: `internal/boardengine/task.go:349-356`.

F5 — MEDIUM, design, D6/D8: the filing write's way past the run-agent task guard is unstated.
- `checkTaskCreation` refuses a new task when `Config.StrandName` parses as a run agent's name. Only `boardcli` sets `StrandName`, from `LYM_STRAND_NAME`.
- The record does not say which strand name `fronttask`'s write carries. The check is spawned by adopt, or by dobby's `on_land`, and inherits the spawner's environment.
- If the plan copies the CLI wiring, a dobby whose environment holds a run agent's name, for example one brought up by a `lym beam start` child of a run's verb, makes the filing refused.
- If the write carries no name, the guard is bypassed with no stated bound. Either way the Decision states neither the bypass nor its bound.
- Evidence: `internal/boardengine/taskguard.go:13-37`, `internal/boardcli/cli.go:161`, `internal/boardengine/config.go:55`.

F6 — MEDIUM, consistency, D7 `running_here`: a halted fix task reports as running.
- `running_here` covers "any other run this machine owns, the fix task included".
- A fix task halted on D9's machine-failure stop, the designed outcome the operator must act on, reports `running_here`. That defeats D7's stated purpose of telling apart whether a run should run.
- Evidence: decision-record D7 state list and Rationale; D6 machine-failure halt.

F7 — LOW, consistency, D8 refusal: only `lym board upsert` is named.
- `upsert-batch` and `merge` take the same entry payload. The model D8 cites, the `beam` refusal, is the engine-level `checkBeamPayload`, which covers every payload path.
- D8 names only `upsert`, so the sole-writer claim in `PATTERN-front-task` is unbound for the other two verbs.
- Evidence: `internal/boardengine/project.go:53-80`; decision-record D8 fourth bullet.

**Tool calls**
1. grep for the board's `beam` field refusal — the manifest lacked where D8's cited model refusal lives.
2. grep for `"beam"` in board code — to locate the engine-level check.
3. `internal/boardengine/project.go:45-80` — to read `checkBeamPayload`'s scope.
4. `internal/boardengine/store.go` and `doc.go` unknown-key rule, plus the `internal/boardengine/replay.go` field table — to attack the tolerant-records constraint for `front` and `on_land`.
