```yaml
lens: bound-breaker
outcome: reported
```

I reviewed the record's claims and bounds through the bound-breaker lens. Two claims break badly, a few more break in narrower ways, and the rest hold.

**Claims attacked**
1. D9's retry ("a Raddle-File interrupted after step 3 … finishes the remaining steps"), checked against what happens once the run's entry is done on origin: **broken**.
2. D9 Bound ("a halt after step 3 … Raddle-File halts with a reason naming the step", which assumes a resume picks the run up): **broken**, same root as claim 1.
3. D9's done marker for step 3 ("the run's entry is done and every filed slug exists"), checked against a filed part that lands and is pruned before the retry: **broken**.
4. D8's second check ("no task or project absent from `board_checked` whose `revised` stamp is later than that point"), checked against a note promoted after the baseline: **broken**.
5. D9 step 3 and D10 claim "one board write and one board commit" in which the raddle store "reaches origin with the filed entries": **not shown to hold**. A board write commits nothing itself; a detached sync commits later.
6. D11 ("a re-invoked or later BeamUp sends none"), checked against a second pawl run of the same beam project: **broken** (minor).
7. D12's Codex output count (`output_tokens`), checked against `reasoning_output_tokens`: **left unsettled**.
8. Constraint fact that `scopeChanged` compares neither Status nor NotifyOnLand: held (`internal/boardengine/runlock.go:191`).
9. Constraint fact that `SetRunStatus` and `MakeSketch` skip a done entry, and `Prune` keeps a done entry carrying `notify_on_land`: held (`board.go:203`, `sketch.go:77`, `store.go:793`).
10. Constraint fact that `StoreDecisionRecord` refuses an absent or done entry: held (`decision.go:46-50`).
11. D9 Bound, that Go's board write carries no strand name so the guard on run agents still holds for the agent's own calls: held. The guard keys on the Board's configured strand name (`taskguard.go:21-23`, `board.go:51`). Raddle-File must open the board without that name, which the record implies.
12. D8, that a status write leaves the `revised` stamp alone so running weighed entries do not fail the re-check: held (`dates.go:21,48`).
13. D7, that a key changed after the first spawn changes no mode: held. A profile or run_settings change reaches only the next Seed-Child (`runsettings.go:77-79`).
14. D2 Bound, that the rule keys on the `discussed` flag and stored record and the flag cannot be set by hand: held (`decision.go:126-138`).
15. The pre-push check of an ended entry, for a raddle pair with no code commits: held. `landed()` uses `ParentHoldsTaskWork` (`prepush.go:71-77`, `prepush.go:206-215`).

**Findings**

- **F1 — BLOCKING, design, D9 (step 3 marks the run's entry done mid-run).** Once step 3's write reaches origin, every origin-aware path treats the run as finished, but steps 4–5 still have to run.
  - The ledger marks the run ended when its entry is done: `Ended: isDone(task)` (`internal/boardengine/replay.go:272`). Origin's verdict then becomes `EntryEnded` (`originreconcile.go:201`, `classifyEntry`).
  - Counterexample: step 4's `lym beam create` fails and the run halts. The batten resume reaches origin, and `resumeRefusal` refuses on `EntryEnded` with "run lym cleanup --apply … instead of resuming it" (`internal/battencli/resumereconcile.go:114-125`). Steps 4 and 5 never run, so the `beam` shape is filed without its beam and the orch gets no notice.
  - A hub reconcile or `lym cleanup --apply` while the pair sits idle tears it down through `teardownEnded` (`internal/pairteardown/originseams.go:155`).
  - A weft replay sets aside every local commit under the run's record directory, including step 5's sent marker and row state. `withoutFinalizeEntered` exempts only runs that entered Finalize (`internal/fabricengine/replay.go:236-251, 305-320, 389-414`), and raddle has no Finalize row.
  - The record's facts cover the run lock, status writes, `MakeSketch` and `Prune`, but none of the origin-reconcile, resume, teardown or replay paths. The AC "Raddle-File interrupted after any step and run again finishes the remaining steps" cannot pass through a real resume.

- **F2 — MEDIUM, design, D9 (step 3 done marker).** The marker also depends on every filed slug still existing, and filed parts are released at once.
  - Counterexample: after a halt at step 4, a filed darn part runs, lands, and Land prunes it.
  - The retry then reads step 3 as not done. It reruns D8, whose free-slug check fails on the other filed slugs, and the run returns to Raddle-Write with its entry already done. If every part was pruned, the slugs are free again and step 3 files them a second time.
  - The run's own entry being done is the step-3 fact; "every filed slug exists" is not a stable second condition.
  - Evidence: `store.go:790-796` (`Prune` removes done entries); the record's D9 lines 121 and 137.

- **F3 — MEDIUM, design, D8 bullet 2 (module-label check keyed on `revised`).** A promote keeps the note's dates, so a note promoted after the baseline slips past the check.
  - `stampDates` exempts the promoted slug from a revision (`internal/boardengine/dates.go:21, 44-47`), and `board.go:119-123` passes `promotedSlug` through.
  - Counterexample: note `x` carries the Sketch's module label, was revised before the baseline, and is not in `board_checked`. The operator promotes it to a task after the proposal passed. Its `revised` stamp is still older than the baseline, so the re-check passes even though a new task now touches the Sketch's packages.
  - A bulk import or upsert that sets `revised` explicitly also keeps an old stamp (`dates.go:48`, `store.go:486` admits `revised` in the payload).

- **F4 — MEDIUM, design, D9 step 3 and D10 ("one board commit", the raddle store "reaches origin with the filed entries").** The board write saves `board.json`, follows the record files, renders, and then only starts a detached `lym board sync --deferrable` whose result it ignores (`internal/boardengine/board.go:160-178`, `spawn.go:39-50`).
  - Nothing in the write commits, and the record does not say the raddle store's new directory is in what that sync stages.
  - My last search for the sync's pathspec in `internal/boardengine` and `internal/boardcli` returned nothing, so it is unproven that the store reaches origin with the entries or in the same commit.
  - The D10 AC (prints the proposal "on another machine once it pulls the board") rests on this unstated fact.

- **F5 — LOW, design, D12 (Codex output tokens).** D12 reads output as `output_tokens`, and the Constraints fact notes that Codex also reports `reasoning_output_tokens`. Neither says whether `output_tokens` already includes the reasoning tokens.
  - If it does not, every Codex session's output is undercounted. That is a fact the design rests on, left open.
  - Evidence: decision record Constraints line 198; `docs/research/codex-review-bench-2026-10-25/report.md:196`.

- **F6 — LOW, consistency, D11 ("a re-invoked or later BeamUp sends none").** The sent marker lives in the pawl run's state.
  - Counterexample: after `lym batten cancel`, which removes the run's records (`runlock.go:84-86`), a new pawl run of the same beam project has fresh state. Its BeamUp finds the proposal in the raddle store and sends the hand-off again.
  - The claim holds only within one pawl run.

**Tool calls**
1. grep `Revised` in `store.go` and `landed` in `prepush.go`: to check whether status writes bump `revised` and how pre-push judges an ended entry; the manifest had neither.
2. grep for writes of `Revised` in `boardengine`: to find the stamping rule.
3. `Read` of `internal/boardengine/dates.go`: the stamping rule itself (`stampDates`), for the promote counterexample.
4. grep of `boardCriticalSection` for commit staging: to test the "one board commit" claim.
5. `Read` of `board.go:96-180`: the critical section's save, record-follow and sync spawn.
6. grep of `checkTaskCreation`, the board's strand-name source and `spawnSync`: to test the strand-name bound and the async sync.
7. grep for the sync's pathspec: returned nothing. This seventh call went one past the 6-call cap.
