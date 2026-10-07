# Card 10 — treadle-triage-retire

**Edit:**
- `internal/treadleengine/run.go`
- `internal/treadleengine/judge.go`
- `internal/treadleengine/judgeverdict.go`
- `internal/treadleengine/roundfiles.go`
- `internal/treadleengine/state.go`
- `internal/treadleengine/runner.go`
- `internal/treadleengine/doc.go`
- `internal/treadleengine/engine.go`
- `internal/treadleengine/engine_test.go`
- `internal/treadleengine/judge_test.go`
- `internal/treadleengine/judgeverdict_test.go`
- `internal/treadleengine/template_test.go`
- `internal/treadleengine/state_test.go`
- `internal/treadleengine/roundfiles_test.go`
- `contracts/stencils/stencils.go`

**Delete:**
- `contracts/stencils/treadle/treadle-template-triage.md`

**Intent:**
D7 for treadle: the asking triage exists only for an asking round, which `Wait` no longer produces, so it goes whole.

- `run.go`: `runRound` retries only after a died or timeout attempt; the `OutcomeAsking` branch, the `triagePath` accumulator and `roundOutcome.TriagePath` go, and the doc comments that mention triage are rewritten.
- `judge.go`: delete `runTriage`, `triageRole` and `triageSkills`.
- `judgeverdict.go`: delete `TriageVerdict`, `TriageRetry`, `TriageGiveUp`, `ParseTriageVerdict` and `parseTriageVerdict`; the file header names the judge verdict alone.
- `roundfiles.go`: drop the `Triage` artifact path; `state.go`: drop `roundRecord.TriagePath` and the triage path from the stale-artifact list.
  Treadle's `state.json` is read with the lenient `state.ReadJSON`, so a record an older binary wrote with `triagePath` still loads.
- `runner.go`, `engine.go` and `doc.go`: drop triage from the retry-policy and spawned-roles descriptions.
- `contracts/stencils/stencils.go`: drop `TreadleTemplateTriage`, its registry entry and the `treadle-triage` role entry, with the stencil file itself.

Tests: delete the triage tests (`judge_test.go`'s triage cases, `judgeverdict_test.go`'s `ParseTriageVerdict` cases, `template_test.go`'s triage rows and `TestRunTriage_ReadsFromDiskAtCallTime`), and rewrite `engine_test.go`'s asking rounds to died or timeout rounds where they test the retry, deleting those whose only subject is triage.
Adjust `state_test.go` and `roundfiles_test.go` for the removed field and path.
Run `contracts/stencils`' registry and role-skill tests to confirm the registry stays consistent.

**ImpactSummary:** Treadle loses its asking-triage spawn, verdict type and stencil; a round retries once on died or timeout only.
