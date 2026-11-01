```yaml
lens: interactions
outcome: reported
```

Task: the interactions lens. I checked pairs of decisions in the machine-switch decision record that act on the same state, and checked the acceptance criteria and constraints against the decisions they touch.

**Pairs checked**
- D4 hold × D6/D8 fix task, on a re-run of `lym adopt`: **broken** (F1)
- D4 hold × dobby's `max_runs` slots × D6/D8 fix-task start: **broken** (F2)
- D5 green-end cancel × D8 front start hold × D6 "never filed twice": **broken** (F3)
- D6 slug cut × D5 cancel × D6 "never filed twice": **broken** (F4)
- D7 `running_here` × D6/D9 halted fix task: **broken** (F5)
- D2 reconcile × D4 owner read from the reconciled Bolt: held
- D4 owner-entry conversion × D4 dobby owner-pause recheck: held
- D5 green lift × D4 holds a second adopt writes: held (the bound is stated)
- D5 try-count reset × D6 count (hand adopt resets, a landing's check does not): held
- D5 `StartFirstPass` of lifted moved runs × D8 front start hold: held, because adopt's direct pass is not dobby's start step
- D8 `root` (no project, parent is the prime) × D5 `BeamUp` of the prime: held
- D9 halt × D8 hold that lifts only on done or removed: held, because a halted front task holding the worktree until a re-run is stated
- D3 state list × D7 states and precedence (`paused` added as an auto-mode assumption): held
- D1 idempotent re-run × D4 bullet 2 (no owner condition): held for moved runs; the fix-task case is F1
- Constraint `PATTERN-dobby-starts-runs` × D4/D6: broken through F1 and F2
- Acceptance D5 "cancels a halted fix task" × acceptance D8 / D8 "hold lifts once done or removed": broken through F3

**Findings**

F1 — BLOCKING, design. D4 bullet 2 against D6 and D8.
- **Summary:** a re-run of `lym adopt` holds a released, not yet started `adopt-fix-<subject>` task, which deadlocks the fix loop.
- **Detail:** D4 holds "each task under an adopted subject that is released with an empty status", with no owner condition and no exception for a front task. If the re-run's check is red, D6 files nothing because "a fix task of the subject that is still open is never filed twice". Only a green check lifts the held fix task, and only a landed fix can make the check green. D7 then reports the fix task as `waiting_for_check`, which contradicts D7's "`running_here` — … the fix task included". Constraint `PATTERN-dobby-starts-runs` is not met: the fix task is never started.
- **Evidence:** `_lym/discussion/decision-record.md:78`, `:131`, `:147`; `internal/dobbyengine/starts.go:52-55` (`readyToStart` refuses a slug the pause record covers).

F2 — BLOCKING, design. D4 against D6/D8 and dobby's start step.
- **Summary:** held moved runs take `max_runs` slots, so the fix task may never start while the runs it must unblock are held.
- **Detail:** `slotsTaken` counts every listed run that is not recorded owned elsewhere, and a held run counts too. `startReady` breaks once `running >= cfg.MaxRuns`. With the template's `max_runs: 3` and three or more moved runs held, the released fix task waits for a free slot. The held runs wait for a green check, and the check needs the fix to land. Neither D4 nor D8 exempts a held run from the slot count or lets a front task bypass it.
- **Evidence:** `internal/dobbyengine/starts.go:158-166` (`slotsTaken`) and `:77-80`; `internal/dobbyengine/template.yaml:1`; `_lym/discussion/decision-record.md:157`.

F3 — BLOCKING, design. D5 green-end cancel against D8 and D6.
- **Summary:** batten's cancel leaves a cancelled fix task as `draft`, not done or removed, so D8's front start hold never lifts and D6 never files again.
- **Detail:** `clearRunClaim` writes `StatusDraft`, and without `--remove` the pair stays, so the slug cannot run again. D8 lifts the start hold only "once it is done or removed". After a green end cancels a halted fix task, the worktree's dobby starts no other task, including the released tasks D5 has just lifted. D6's "still open" fix task, now a draft, also blocks every later filing. Acceptance D5 ("cancels a halted fix task") and acceptance D8 ("halted or cancelled front task runs none") both pass while the beam stays wedged.
- **Evidence:** `internal/battencli/cancelcmd.go:491-503` (`clearRunClaim` writes `StatusDraft`); the `cancelCmd` help text ("The pair is kept by default, and `lym batten cancel <slug> --remove` must run before the slug runs again"); `_lym/discussion/decision-record.md:114`, `:161`.

F4 — MEDIUM, design. D6 slug rule against D5 cancel and D6 "never filed twice".
- **Summary:** cutting the subject so the slug fits lets two subjects share one fix slug.
- **Detail:** `MaxSlugLength` is 32 and `adopt-fix-` takes 10 characters, so two beams sharing their first 22 characters map to one slug. The effects:
  - Subject A's failed check sees B's open fix task as its own and files nothing.
  - A's green end cancels B's halted fix task.
  - A filing upserts onto B's entry and moves it into A's project.
- **Evidence:** `internal/boardengine/task.go:350`; `_lym/discussion/decision-record.md:128`, `:131`, `:114`.

F5 — MEDIUM, consistency. D7 states against D6/D9.
- **Summary:** a fix task halted on a machine-failure stop reports `running_here`.
- **Detail:** D7 sends "any other run this machine owns, the fix task included" to `running_here`. D6 and D9 make a halt the designed outcome of a machine failure, and that halt is the state the operator must act on. The closed state set has no state that tells a halted run from a running one, so the report misstates the very run the operator has to see.
- **Evidence:** `_lym/discussion/decision-record.md:147`, `:134`, `:170`.

Tool calls: none; my inherited context already held every file and member these findings cite.
