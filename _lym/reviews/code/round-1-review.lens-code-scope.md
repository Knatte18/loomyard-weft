```yaml
lens: code-scope
outcome: reported
```

Task restated: code-scope lens — map each change in the diff to its decision, and each decision to its changes.

## Change → decision

| Change | Traces to |
|---|---|
| `internal/testreach`: `reader.go`, `index.go`, `mapread.go`, `fresh.go`, `doc.go` | D1 |
| `internal/testreach/changedfuncs.go` | D2 |
| `internal/testreach/changedlines.go` | Forced by D9 and D11 (changed lines of a diff) |
| `gatecli/reach.go`, `reachreport.go`, the `run.go` reach step, `gateslot` `ReportReach` | D3, D4 |
| `gatecli/testmap.go`: `deadline` and `mapBudgetReason` | D3 (`reach_budget_min`) |
| `gatecli/testmap.go`: `writeIndex` | D1 |
| `verifytree/reach.go`, `shuttleengine` `WriterOnly` and `RerunMark`, `gatefixer/rowgate.go`, `shedrecipe` `VerifyReach` and `entries_gate.go`, `weaverengine` `ReachPending`, `loomcli/wiring.go`, `envkit` `nilLegal` | D3 and the reach-turn bypass entry (arbiter); `envkit` is forced |
| `shedadapters/rubric.go`, `bouncer.go`, `shedrecipe` `ReachReport`, `loomcli/reachnote.go`, the code-review rubric stencil, `contracts/stencils/rubric_test.go` | D5 |
| `gatecli/reachverb.go`, `fabricengine` `FirstParentCommits`, the `cli.go` verb wiring | D6, D8 |
| `gateslot/triage.go`, `TriageLinePrefix`, `gatecli/triage.go`, `failurereport.go`, `counters.go` | D7 |
| `runplan` `Triage`, `loomshed/verifygate.go`, `weaverengine/strand.go` `failureReport`, `websterengine/verifygate.go` | D7 (every fixer's report) |
| Prime treated as having no parent branch (`addTriage` merge-base closure) | "Triage in the prime follows D7" entry |
| `testcover/linemutant.go`, the `mutant.go` `callee` field and `ownBlocks`, `gatecli/mutate.go`, `mutatecheck.go`, the `redundantmutants.go` `site` and `profileBlocks` changes | D9, D11 |
| `gateslot/config.go`, `template.yaml`, `tiers.go`, config tests | D12 |
| `PATTERN.md`, `pattern/PATTERN-coverage-map-reader.md`, `pattern/PATTERN-test-speed.md`, `pattern/PATTERN-gate-slot.md` | PATTERN changes. `PATTERN-gate-slot` was not declared; it is forced by the reach refresh holding a gate slot. |
| `cmd/lym/coveragemapreader_test.go` | Enforces the declared `PATTERN-coverage-map-reader` entry (D1, D2) |
| `docs/overview.md` and the package `doc.go` edits | Forced by the verbs and steps above |
| `weaverengine/testmain_test.go` `HermeticGitEnv` | Forced: the new strand integration test spawns git |

No change traces to nothing, and none builds D10. Config accepts only `reach`, and a test pins `mutation` as refused. The trial and the `mutation` report step are not built, as the arbiter entry requires.

## Decision → changes

- D1: `testreach` reader and index, plus `writeIndex`.
- D2: `changedfuncs.go`.
- D3: the reach step, the refresh and its budget, `reach.json`, the reach turn, the re-run mark and the `WriterOnly` bypass.
- D4: `ReachReport.NoteLines`, the caveats, `StaleLinkers`.
- D5: the `reach_report` marker and `ReachReportMatches`.
- D6: the function form of `reachverb.go`.
- D7: the triage lines and the fixer-report plumbing.
- D8: the test form of `reachverb.go` and `FirstParentCommits`.
- D9: `PlanLineMutants` and `diffMutation`.
- D10: none, by design.
- D11: `mutate.go` is built. The board note is not visible in the diff (see finding F1).
- D12: the template, config keys and `ReportReach` acceptance.
- Arbiter entry (mutation order): satisfied.
- Arbiter entry (reach-turn bypass): satisfied.
- Parent entry (prime triage): satisfied.

## Findings

**F1 — LOW, scope.** The D11 board note asking for the mutation trial is not evidenced in the committed diff.
- Location: `_lym/plan/13-gate-mutate.md:135` (the card asks for it via `lym board upsert`, `kind: note`).
- Evidence: the diff has no board change. `_lym/landing/summary.md` is absent, so there is no change-description claim that the note was filed.
- The note goes through `lym board` into the records repository, so the diff cannot show it. The handler should check the board, for example with `lym board get`. If no note exists, raise it as MEDIUM scope.

**F2 — NIT, scope.** `pattern/PATTERN-gate-slot.md` changed, though the declared PATTERN changes list only `PATTERN-coverage-map-reader` and `PATTERN-test-speed`.
- Location: `pattern/PATTERN-gate-slot.md`, new bullet "Inside the refresh of a task worktree's own anchor maps".
- It is forced by D3's slot use and consistent with the record's constraints section, so I recommend accepting it.

No other scope findings.

Tool calls (1): I ran `grep` on `_lym/plan/13-gate-mutate.md` and `00-overview.md` for "board", because the diff cannot show whether the D11 board note was filed.
