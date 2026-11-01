```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked each pair of decisions that act on the same state, key, command or ordering, including their failure and restart paths.

**Findings**

1. **BLOCKING** — D5 × D6 × D9 × D13: an ordinary resume of `Parts-Plan-Write` ends the step as soon as any part writer has finished.
   - D13 gives the row the reinvoke interrupt policy, so an interrupt goes through Probe and then Resume.
   - Probe removes a finished seat whose pane is still up and reports it as not found.
   - D5 keeps today's rule that a required seat Resume does not find ends the step. Every writer is required, and `internal/seatengine/resume.go` calls `ender.end(lost)` for any such seat, finished or not.
   - Under D9 the chair waits for all writers, so most writers will have finished by then. A plain pause and resume, or a driver restart, then halts the run (Stuck with no `on_stuck`, D6), not only a real crash.
   - D6's bound, "a crash that loses any writer's strand ends the step on resume", understates this.
   - D5 also says a finished writer "keeps answering the chair", but after Probe removes its strand it cannot, so D9's fix-and-rejoin loop also breaks after any resume.
   - Evidence: `resume.go` lines 111–124, and `doc.go` "a done one kept by its pane … is removed by the probe itself … and answers not found".
   - Fix: a required seat whose outputs all exist and whose result was accepted should count as finished on resume rather than lost, and D5 should say so.
2. **MEDIUM** — D3 × D7 × D9: a part card cannot depend on something a shared-spot card creates.
   - D3 gives shared spots no exports. D7 counts only the exports of a part's dependencies as declared. D11 tells writers to leave shared spots to the chair.
   - D9 places shared-spot cards "before the first card that uses it", which assumes part cards do use what those cards create, such as a new config key or Env seam.
   - In that case `handle-dangling` or `path-missing` fails at the part gate, and the writer cannot fix it, so its gate runs out and the step ends.
   - Fix: give shared spots exports in the parts file and count them as declared in the part check.
3. **MEDIUM** — D3 × D11/D7: the parts check refuses "a shared-spot path that is also inside a part's area", so no part can own a package that contains a shared spot.
   - Areas are packages and files, and a shared spot is usually a file inside a package some part has to change. Example: `internal/shedrecipe/registry.go` sits inside `internal/shedrecipe`, where a part writing a new Constructor row works.
   - Every split of that kind is refused, or pushed into one part per package.
   - Fix: refuse only a shared-spot path listed exactly as a part's own file, or let a part's area name a package minus its shared spots. D7's target-level check already keeps writers off shared spots.
4. **MEDIUM** — D1 bounds × D1/D6/D8: two claims in D1's bound are false.
   - "The gate is read once, on entry; a flag changed after a run passed `Drift-Baseline` does not move it." Each row evaluates its own gate when the run enters that row. If the flag changes between `Parts-Write` and `Parts-Plan-Write`, the run skips `Parts-Plan-Write`, runs `Plan-Write`, and leaves the parts file committed but unused.
   - "A goto onto either branch runs that branch's rows whatever the flag says." With the flag off, `goto --to Parts-Write` runs `Parts-Write`, then `Parts-Plan-Write`'s gate skips it and `Plan-Write` runs.
   - Also with the flag off, `goto --to Parts-Plan-Write` joins a plan, then `Plan-Write`'s gate (`!= true`) holds, so `Plan-Write` runs and its rotator archives the joined plan.
   - Evidence: `shedengine/entry.go` checks `def.EntryGate` per row unless that row is `EnteredRow`, and `goto.go` sets `EnteredRow` only for the goto target.
   - Fix: restate the bound as it actually behaves.
5. **LOW** — D4 × D6: where the archives go is unclear. D4 archives "a stale `_lym/parts/` into an archive directory beside it", and D6 archives part directories and `join.md`, apparently inside `_lym/parts/`. If D4's archive lands inside `_lym/parts/`, each later archive of the whole directory nests the earlier ones; `NewPlanDirRotator` avoids this by moving files only. Name each archive's location and say that only non-archive entries move.
6. **LOW** — D5 × D9/D11: when a finished writer fixes its part at the chair's request, no new done line is queued, so a chair that ended its turn is not woken. D11 should require the writer to reply to the chair's creel message once the fix passes `--part`.
7. **LOW** — D8 × D9: the chair must settle every part before inserting any shared-spot card, because a rejoin discards those cards (D8's bound). D9 says to join "once every part stands" but never says not to rejoin after inserting, or that a later mismatch means redoing the insertions. One sentence in D9 or D12 would close it.

**Pairs checked**
- D1 entry gates × D2 `plan_split` as a bool with `!= true`: held; `configreg.CheckSettingKey` serves `SettingKnown`.
- D1 bound × per-row gate evaluation: broken (finding 4).
- D1 goto bound × D6/D8 rotator and join: broken (finding 4).
- D1 × constraint "Plan-Write's edges unchanged": held; the entry gate is not an edge.
- D2 `parts_max` × D3 check: held.
- D2 `plan_lenses` × D9 `plan` gate's Forked closure: held; no fork authorization.
- D3 shared-spot rule × D7 target rule × D11 area: broken (finding 3).
- D3 no shared-spot exports × D7 declared set × D9 placement: broken (finding 2).
- D3 decision-id coverage × the record's id-less "Added after Discussion" entries: held.
- D4 archive × D6 archive: unclear (finding 5).
- D4 one-part fallback × D6 table and D8 join: held.
- D5 required writer × D6 Stuck halts the run: held, as intended.
- D5 resume rule × D13 reinvoke × D9 waiting: broken (finding 1).
- D5 done line × D9 fix loop: weak (finding 6).
- D5 "advisors only or writers only" × D6 table: held.
- D6 rotator × D8 replacing the plan directory: held.
- D6 commit × D9 `join.md` as last output: held.
- D7 excluded checks × D9 `plan` gate running the full set: held.
- D7 `pattern-changes-mismatch` excluded × D9 chair copying `## PATTERN changes`: held.
- D8 rerun discards chair edits × D9 order of work: held only if the stencil says so (finding 7).
- D8 export check × D3 exports, including re-sign and delete: held.
- D8 numbering from 1 × rework generations: held; Plan-Burler-Fix writes rework generations, not this row.
- D9 output `join.md` × done meaning outputs exist: held.
- D12 block markers × D6 per-seat values: held if filled per seat; the record's Notes say so.
- D13 timing phase × D1 skipped rows: held.
- Acceptance criteria: D5's criteria never cover resume (finding 1); all others match their decisions.

**Tool calls**
- grep of `internal/loomcli/wiring.go` and `internal/seatengine/resume.go`: to confirm how entry-gate settings are wired (`SettingKnown`) and where resume handles a required seat; my context had neither.
- Read of `internal/seatengine/resume.go` lines 105–139: to confirm that a finished required seat Probe does not find ends the step; my context had only the doc summary.
