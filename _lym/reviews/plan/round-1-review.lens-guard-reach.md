```yaml
lens: guard-reach
outcome: reported
```

Lens: guard-reach. For each guard, refusal, branch or test row the cards add, I traced whether a real entry point can reach it and whether the guarded site recurs somewhere the cards leave unguarded.

**Per guard: reachable or not**
- Card 2: `LoadConfig` and `ResolveLadder` refusals are reachable through a layer file or a row build.
- Card 6: `Active` refusals.
  - The corrupt-file and setting-value refusals are reachable.
  - "Both begun" is reachable only through planted files or a webster verb run by hand (F6).
  - `restorableVersion`'s conditions are reachable.
- Card 9: the gate config with no `card` or `batch` key is reachable, though only at spawn time, mid-run. Every `classifyAttempt` verdict is reachable.
- Card 12:
  - `ErrRunBusy` is reachable.
  - The failed stop and the failed state write are reachable.
  - The plan-conflict Stuck is reachable.
  - The Stuck on a failed commit at Done is reachable, but no test row reaches it (F4).
- Card 13:
  - Every resume step is reachable: stop recorded strands; probe attempts in flight; a live fixer record, and a fixer number with no record.
  - The probe-error Stuck is reachable.
  - The `reenter` branch is reachable, but also in a state it does not guard: a `StopRecord` that is never cleared (F1).
  - The fixer spawn and retry branches depend on how the outcome is recorded (F2).
- Card 14:
  - Steps 1, 3, 4, 5, 6 and 7 are reachable.
  - Step 2 is reachable in a state it does not guard: no Weaver run, with pending plan edits (F5).
- Card 15: the outside-hub refusal is reachable through `clihelp.ExecuteIn`.
- Card 16: the conflict claim, the busy run and the fault mappings are reachable.
- Card 17: the guard refusals are reachable. The begin site recurs unguarded in the webster verbs (F6).
- Cards 19, 20 and 22: the seam refusals are reachable through a corrupt state file.
- Card 21: the held-back branch is reachable after a failed stop or a crash. The row "corrupt Webster state still notifies" is reachable for a batch strand.
- Card 23: the row "corrupt Webster state still warns of Weaver" is not reached as the card writes it (F3).
- Card 25: the fallback row is reachable, since it sets the setting.

**Findings**

F1: MEDIUM, design. Cards 12, 13 and 5 never clear `StopRecord`.
- Card 13 runs `reenter` "when `StopRecord` is set", and its comment says "a call after a plan-conflict Stuck finds none". No card clears the record once a re-entry succeeds, or once the next batch begins.
- So after one batch stop and its successful re-entry, a later plan-conflict Stuck, or a crash re-step, reaches `reenter` again. It re-runs the post-batch checks on an old batch, or gives it a last-rung attempt.
- Evidence:
  - `_lym/plan/13-run-reentry-and-resume.md:28` (`reenter` runs when `StopRecord` is set)
  - `_lym/plan/05-weaver-state.md:23` ("the row's last … stop")
  - `_lym/plan/12-run-loop.md:48` (records a stop, never clears one)

F2: MEDIUM, consistency. Cards 11 and 12 leave the fixer's terminal status unwritten.
- `fixerStore.Live` returns the newest fixer whose `Record` is set and not terminal. `gatefixer.Await` never marks a record terminal: "the caller does, through its store".
- Card 12 only "records its outcome when `Await` returns terminal". It never says the outcome is saved as `Record.Status` through `fixerStore.Save`.
- If it is not saved, `SpawnOrAttach` returns the ended record as live and spawns nothing. The next fixer, and a lost fixer's same-rung retry, then cannot be reached.
- Evidence:
  - `internal/gatefixer/await.go:39` ("Await never marks rec terminal")
  - `internal/gatefixer/spawn.go:89-95` (the live record is returned and nothing is spawned)
  - `_lym/plan/11-plan-verify-and-fixer.md:43`
  - `_lym/plan/12-run-loop.md:44`

F3: MEDIUM, consistency. Card 23's corrupt-Webster row cannot reach its expected warning as written.
- A corrupt Webster `state.json` makes `Active` refuse, so `ActiveOrSetting` falls back to `tapestry.implementer`. The default is `webster`.
- With the default, the warning reads Webster's in-flight state, which errors on the corrupt file, so no warning is emitted.
- The row reaches "still warns of Weaver's run" only if the test sets the setting to `weaver`, and the row does not say so. Card 25's matching row does say so.
- Evidence:
  - `_lym/plan/23-merge-in-warning.md:31`
  - `internal/fabriccli/merge_verbs.go:28-32` (an error degrades to no warning)
  - `_lym/plan/25-batten-fixer-timeout.md:22` (contrast)

F4: LOW, consistency. Card 12 prescribes a Stuck when the final commit at Done fails, naming the re-step, but no `TestRun` row reaches it. Card 10's commit-failure row covers only the per-batch commit.
- Evidence: `_lym/plan/12-run-loop.md:46` against the `TestRun` list at `:51-63`.

F5: MEDIUM, design. Card 14's step 2 fires where no earlier reset ran.
- With no state file, `RecordPending` reports uncommitted changes under `_lym/weaver` or the plan directory, and the reset commits them.
- That state also occurs when no Weaver run ever began: a Webster run, or plan edits made under review. `lym weaver reset` then commits unrelated plan edits as a "finished reset".
- The branch's guard cannot tell a failed reset's leftovers from other pending plan changes, and the card states no bound.
- Evidence: `_lym/plan/14-weaver-archive-and-reset.md:25` and its `RecordPending` definition at `:22`.

F6: LOW, scope. The D1 guard sits only on the two implementer rows (card 17).
- The site it guards, beginning batches of the other implementer, recurs unguarded in `webstercli`'s `begin-batch` and `run` verbs, which load Webster's state directly and never consult the seam. "Both begun" can arise only through those verbs or planted files.
- D1 scopes the guard to the row, so the risk is bounded to manual operator verbs.
- Evidence:
  - `internal/webstercli/beginbatch.go:145` (direct `LoadState`, no seam)
  - `_lym/plan/17-implementer-guard.md:30-31`

Tool calls:
1. Grepped `internal/loomcli/wiring.go` for how `Env.Setting` is filled: the manifest did not show the setting lookup, which bears on whether the seam's setting matches the row's.
2. Grepped for the `settingLookup` definition (`configreg.SettingLookup` over the location's config dirs), to confirm card 18's seam setting can share it.
