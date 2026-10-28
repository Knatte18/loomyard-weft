```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran the Darn-Write prior-work, findings-path and re-prompt changes against concrete inputs. Two findings, both LOW.

**Per function or branch (case run, verdict)**

- `renderPriorWork`, `found=false` with a stale `FindingsPath`: returns "". Correct.
- `renderPriorWork`, `found=true`, `Outcome=done`, no findings: four lines, no findings line. Correct.
- `renderPriorWork`, `found=true`, `Outcome=stuck`, findings kept: all five lines. Correct.
- `renderPriorWork`, `found=true`, `Outcome=yield`, `skipped` or `goto`: reads "That spawn ended skipped." and still claims committed work. See finding 1.
- `latestDarnOutcome`, absent status file (lock open fails with not-exist): returns not found. Unchanged and correct.
- `latestDarnOutcome`, lone Darn-Write reroute entry: skipped, returns not found. Correct.
- `latestDarnOutcome`, resumed run (state running, error cleared, Stuck entry, file present): returns `Stuck` with `FindingsPath`. This is the fixed case, correct.
- `latestDarnOutcome`, findings file absent: not-exist is swallowed and the path is left empty. Correct.
- `latestDarnOutcome`, `findingsPath` empty: `os.Stat("")` reports not-exist, so the path is empty. Correct.
- `latestDarnOutcome`, `os.Stat` fails with another error (ENOTDIR, EACCES): returns the error. `Call` wraps it and returns it, so the run halts with an engine error rather than a Stuck. This follows the "fault is not Stuck" rule. Correct.
- `latestDarnOutcome`, stale file: `KeepGateFindings` removes the file on a call whose gate passed (`multillm.go:76-78`). A later Done entry therefore leaves no stale path. Correct.
- `darnDeps` path: `shedrun.GateFindingsFile(l, runID, loomshed.NameDarnWrite)` is the same function `Env.GateFindingsFile` calls (`wiring.go:554`), so the path matches the one the row writes. This holds only if `row.Name` equals `NameDarnWrite`. Correct for the shipped recipe.
- `DarnGateReprompt`: the rendered line is single-line and non-empty, which is what `validateSendText` requires. It names the findings path and the decision record. Correct.
- `DarnGateReprompt`, empty `decisionRecordPath`: renders "...in  forbids..." with no error. See finding 2.
- `darnWriteEntry` loop, `gate` nil (no `gates` key): the loop does nothing and `table.Gate` stays nil. Correct.
- `darnWriteEntry` loop, `gate` set: `resolveRowGateSpec` builds a fresh slice with `make`, so the mutation touches no shared or cached state. `table.Gate = gate` overwrites any gate on the `env.DarnTable` table, so every entry carries the line. Correct.
- `darnWriteEntry`, `env.DecisionRecordPath`: set in the full `wire` Env literal (`wiring.go:571`), so it is non-empty in production. `darnWriteEntry` does not check it.

**Findings**

1. LOW, design. `internal/loomshed/darnwrite.go` (`renderPriorWork`, the `That spawn ended %s.` line) with `internal/loomcli/darnwiring.go:129-133`. `latestDarnOutcome` skips only `OutcomeReroute`, so a Darn-Write history entry with `skipped`, `yield` or `goto` (`shedengine/producer.go:26-40`) renders the "work is committed on the task branch" note and "ended skipped". A skipped row has no earlier work, so the note would be false. I could not confirm that a Darn-Write row can record such an entry, so this rests on the outcome set alone.

2. LOW, design. `internal/shedrecipe/entries_darnwrite.go:28-30`. `darnWriteEntry` passes `env.DecisionRecordPath` into the re-prompt without checking it is non-empty or absolute. An unwired Env yields a line that names no decision record and reads "in  forbids". The Darn-Write path in `internal/shedrecipe/entries_gate.go:140` checks it with `requireAbsRoot`, but only in the "discussion" gate. Production wiring always sets it, so this only bites a mis-wired Env.

Not a finding: "bar one a decision ... forbids fixing" is singular but reads as covering every forbidden finding. This is wording, not a defect.

**Tool calls**
1. `sed -n 117,146p internal/loomcli/darnwiring.go`: the manifest only had the diff hunks, and I needed the skip filter before the diff context.
2. `grep KeepGateFindings` in `internal/shedadapters`: to confirm the findings file is removed on a passing gate, which the manifest did not show.
