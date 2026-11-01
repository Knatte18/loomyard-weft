```yaml
lens: interactions
outcome: reported
```

Restating my task: the interactions lens, covering pairs of the record's decisions that act on the same state, and acceptance criteria and constraints checked against their decisions.

**Pairs checked**

1. D5 (writer's failed end ends the step) × D6/D1 ("a Stuck from a failed writer halts the run"): **broken**.
2. D5 (resume keeps finished parts) × D6 (fresh-spawn prep archives every part directory) when the chair is lost: **broken**.
3. D5 (writer's Accept runs on outputs at resume) × D7 (Accept is the part check) × the existing `Accept func(shuttleengine.Result)` seam: **broken (minor)**.
4. D5 (chair's hold deferral counts writers) × D11 (a done writer fixes its part on request, the chair waits): **broken**.
5. D3 (shared spots carry no dependency field) × D8 (join orders by parts-file order) × D9 (chair places shared-spot cards by position): **broken**.
6. D1 (settled-verdict refusal names the writer from history) × D1 (goto onto `Plan-Write` in a split run): **broken (minor)**.
7. D4 archive × D6 archive, both under `_lym/parts/`: held. D4 moves everything but archives; D6 keeps `parts.md`; neither nests.
8. D4 commit × D6 commit, both pathspecs holding `_lym/parts/`: held.
9. D7 part check × D8 join recheck: held. The join reruns the same check, so a part edited after its gate is checked again.
10. D8 join-once × D9 shared-spot cards: held. D9 forbids a rejoin once the next step begins, and D8's bound states the discard.
11. D3 shared-spot exports × D7 (count as declared for parts listed as needing them) × D9 (chair creates them): held.
12. D9 (chair fixes a non-answering writer's part) × D8 (join reruns the part check over that fix): held.
13. D2 `parts_max` × D4 one-part fallback: held.
14. D2 `plan_timeout_min` for both rows × D5 (writer waits quietly on a ruling): held. The bound is the timeout.
15. D13 reinvoke × D5/D6 probe-and-resume: held while the chair is live (see pair 2 otherwise).
16. D12 plan-format block × D11/D9 (part writer's `plan_dir` is its part directory, per Notes): held.
17. D7 `--part` × Constraint PATTERN-loom-contract (refuses by rows): held. D7 omits it, but the Constraints line covers it.
18. Acceptance D6 × D6/D1 failure path: broken, the same defect as pair 1. No acceptance bullet pins what a failed writer does to the run.

**Findings**

1. **MEDIUM — design** (D6 "A Stuck from a failed writer or an exhausted gate halts the run (D1)"; D1 "Neither new row carries `on_stuck`: a Stuck halts the run").
   Summary: under today's engine, a required seat ending the step leaves the run failed with an agent-died error, not Stuck. D5 makes writers required, so D6's claim about the failure path is false.
   Evidence:
   - `internal/seatengine/engine.go:361-370`: on `ender.signal`, `join` stops the chair and waits it out.
   - `internal/shuttleengine/wait.go:1789-1791`: a stop-marked run with missing outputs records stopped.
   - `internal/shedadapters/multillm.go:169-183`: `mapOutcome` turns Died into an error wrapping `ErrAgentDied`. Nothing in it reads `EndedStep`, and the grep for `EndedStep` outside seatengine found no consumer.
   - So the row returns an error, not `shedengine.Stuck`. The halt reason and the operator's way back in D1 rest on that false premise, and no acceptance bullet pins the outcome.
   - Only an exhausted chair gate reaches Stuck (`multillm.go:157-162`).

2. **MEDIUM — design** (D5 resume rule × D6 Bound "a resume keeps every finished part").
   Summary: the resume path that keeps finished parts runs only when the chair is live. A lost chair takes the fresh path, where D6's own preparation archives every finished part. D6's bound names only a lost writer.
   Evidence:
   - `internal/seatengine/resume.go:52-60`: `Probe` with no live chair stops every writer and returns an empty table.
   - `resume.go:87-89`: `Resume` refuses without a live chair.
   - `internal/shedadapters/multillm.go:113-127`: the fresh path runs `prepareFreshSpawn`, which D6 sets to move "every part directory and `join.md`" into an archive.
   - So a chair crash or timeout discards every finished part. The D6 bound should state this; the record is silent.

3. **LOW — design** (D5 "counts as finished when all its outputs exist and its Accept passes on them"; D7 "the writer seat's Accept").
   Summary: Accept is `func(shuttleengine.Result) error`, judged on a terminal result. A writer the probe did not find at resume has no result, and the record does not say what Accept receives there or that the seam changes.
   Evidence: `internal/seatengine/table.go:89-91`; `resume.go:117-124`, where a lost seat carries no terminal result.

4. **LOW — design** (D5 "The chair's hold deferral counts writer seats as it counts advisors" × D11 "after done, keep answering the chair; fix its own part when asked … reply to the chair once it passes").
   Summary: deferral never counts a seat whose run has ended. A chair waiting on a done writer's requested fix is therefore not deferred, and its held turn notifies the parent. That contradicts D9's planned wait for fixes.
   Evidence: `internal/seatengine/work.go:33-46`: `outstanding` skips `run.ended()`.

5. **MEDIUM — design** (D3 shared spots × D8 ordering × D9 placement).
   Summary: a shared spot lists the parts that need it and its exports, but not what it uses. Nothing checks that a spot's cards can be placed after what they use and before what uses them. A spot needed by part 1 that uses an export of part 2 has no valid position, and no check or refusal catches it before the chair's `insert-card` placement.
   Evidence:
   - Record D3: the parts check's refusal list has no shared-spot ordering case.
   - Record D9: placement rule.
   - Record Open risks line 3 names the risk, but no Decision states the bound.
   - `planparser/validate.go:1120` (`uses-later-target`) is the only catch, and only where refs show it.

6. **LOW — consistency** (D1 "names the plan writer this run ran, read from its history: `Parts-Plan-Write` when `Plan-Write` was recorded Skipped").
   Summary: after D1's own way back (`goto --to Plan-Write` in a split run), history holds both a Skipped and a later run `Plan-Write` entry. The rule as written still names `Parts-Plan-Write`, though the latest plan came from `Plan-Write`; the latest entry should decide.
   Evidence: record D1 paragraph 3; `internal/shedengine/entry.go:57` (Skipped appended to history); `internal/loomcli/settledguard.go:97-104` (today's `segmentWriter`).

Tool calls: none; my inherited context held everything this lens needed.
