# Support log — agent-self-reflect

## Interview

Autonomous session: no operator answered.
Each turn below is a question the design had to settle, the evidence read, and the pick.

1. **Is the gate under `step` the whole problem?**
   Read `reflectFrictionRow` (`internal/loomcli/run.go`): it reflects only when armed for `run`.
   Read `internal/shedverbs/spec.go`: `PostRun` runs only under `run`, so under `step` there is neither a blocked-path reflection nor Tier 1 anomaly filing.
   Pick: remove the gate at the terminal row (D1) and add a halt path under `step` (D2).
2. **When should a blocked run's friction be filed?**
   #350 and #352 were both Webster blocks the hub filed by hand at the halt.
   Reflecting only at `done` would file them only if the run later finished, after the hub had already filed them.
   Pick: reflect at the halt under both verbs, after a Go-written halt note (D2).
3. **How do Merriam's refusals become notes?**
   Read `contracts/stencils/friction/friction-directive-orchestrator.md`: Merriam's note is optional and agent-written.
   Found `writeTriageFrictionNote` in `internal/websterengine/triage.go`: Go already writes a note mechanically.
   Pick: Go writes a note at every webster verb refusal except `validate` (D3).
4. **Which driver records feed the reflection, and who files?**
   Read `plugins/ly/skills/ly-drive/SKILL.md`: repair records sit in untracked scratch, drive reports in `_lyx/shed/<slug>/drive-reports/`, and the driver drafts its own self-report.
   Two filers would duplicate issues; reading scratch and drive-reports from Go would need a second archive mechanism.
   Pick: repair records go into `friction_dir`, the driver files nothing, drive reports are not a source (D4).
5. **Does the reflection's wait under `step` need ly-drive changes?**
   ly-drive waits on the step job's exit with no timer; a killed job is an interrupted invocation, and Friction-Reflect's policy is `reinvoke`.
   That policy's premise (`step` spawns nothing) breaks once the row reflects under `step`, and a re-invocation could spawn a second filer.
   Pick: make `Reflect` re-entrant through attach and a covered-notes record (D5); no ly-drive wait text.
6. **Self-reflection sessions for Merriam and the driver?**
   With D2–D4, every intervention named on the board lands as a note.
   Merriam's friction comes from its existing optional orchestrator directive plus D3's refusal notes, the driver's from D4.
   Pick: no extra self-report sessions (Scope Out).
7. **Ordering against batten's teardown.**
   Read `contracts/recipes/batten-recipe.yaml`, `internal/battenshed/doc.go` and commit `676d9ec03`: done persists after Friction-Reflect, notes are committed records, and the weft tip is tagged under `archive/` at teardown.
   #274's deletion was closed by those durable records and the archive tag; what remained was that nothing read the notes, which D1 and D2 fix.
   Pick: no batten change (Constraints).
8. **Dedupe against open issues?**
   The board entry records the operator's statement that filtering happens at intake, as in millhouse (`mill-self-report --auto` files all candidates).
   Pick: no dedupe; the reflection only merges duplicates within its own notes and adds provenance (D6).
9. **Tier 1 under `step`?**
   Considered in light of "no gate"; see Rejected alternatives.
   Pick: out of scope.

## Rejected alternatives

- **Keep ly-drive's self-report and only drop the approval gate.**
  Two filers per run (driver and reflection) would file the same repairs twice, with nothing to dedupe between them.
- **Reflection reads drive reports and `<scratch_dir>/repairs/` in place.**
  Those files are never archived after reflection, so each later reflection in the task would refile them; it would need a second archive or marker mechanism beside the friction directory's.
- **Merriam writes refusal notes by a stencil instruction.**
  Depends on the agent remembering; the refusal site in Go is the one place that sees every refusal.
- **Reflect only at the terminal row.**
  Files a blocked run's friction only after a person resolves the block, which is when the hub has already filed by hand; a run never resumed files nothing.
- **Per-agent end-of-session self-report (a `mill-self-report` analogue).**
  A second filing path that duplicates the reflection's issues; the sources already cover the board's named interventions.
- **Dedupe against open issues from the reflection agent.**
  Needs `gh` reads or comments outside `lyx selfreport`, and the operator placed filtering at intake.
- **Friction-Reflect returns Stuck and retries on a failed reflection.**
  Would let an optional bookkeeping agent hold a landed run open; the recipe deliberately keeps it always-Done.
  The parent notification on `friction: failed` (D4) surfaces the failure instead.
- **Tier 1 anomaly filing under `step` in the same change.**
  Tier 1's triggers (escalation, exhausted budget, producer failure) are the same halts D2 notes and reflects on, so running both would file each halt twice; its crash-resume trigger relies on `run`'s entry observation, which `step` does not take.
- **Change batten's teardown to archive or copy friction notes.**
  Already done by `676d9ec03` (records committed, weft tip tagged); the remaining gap was that nothing read the notes.
- **Go parses a filing status out of the reflection report.**
  Writing the report only after filing succeeded (D6) gives Go the same signal through shuttle's existing file contract, with no report format to keep in sync.
- **A dedicated GitHub label for reflected issues.**
  No intake exists to consume it yet; provenance in the body is enough for a later intake to match on.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Does Friction-Reflect reflect under `step`? Resolved, auto-pick: yes (D1). The operator's no-approval decision overrides the gate's reason.
- Q2 Reflect at halts under `step`? Resolved, auto-pick: yes, after a Go halt note, for `blocked` and `failed` (D2). Extends `run`'s blocked-only behaviour to `failed` for symmetry.
- Q3 How are webster refusals recorded? Resolved, auto-pick: Go-written note at every refusal except `validate` (D3).
- Q4 Driver records and filing? Resolved, auto-pick: repair records into `friction_dir`, driver files nothing, drive reports not a source (D4).
- Q5 ly-drive tolerance of the reflection wait? Resolved, auto-pick: no skill text; re-entrant `Reflect` covers a killed job (D5).
- Q6 Self-reflection sessions? Resolved, auto-pick: none (Scope Out).
- Q7 Dedupe against open issues? Resolved, auto-pick: none; left to intake, per the operator's statement on the board (D6).
- Q8 Failed terminal reflection? Resolved, auto-pick: done still persists; parent notified on `friction: failed` (D4); listed as an open risk.
- Q9 Tier 1 under `step`? Resolved, auto-pick: out of scope (Rejected alternatives).
- Q10 New CONSTRAINTS invariant? Resolved, auto-pick: Friction Capture invariant (D7).
- Environment note: every `lyx` call logs stencilstore drift warnings (board copy older than the worktree source, "run a production deploy"); they did not block anything and were left alone.
