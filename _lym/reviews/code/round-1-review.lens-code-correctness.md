```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens, covering each changed function or branch, the case I ran, and any defect.

**Walk-through list**
- `Accept`, operator session, scope partial and full: ran status running at Drift-Verify, mark open, `OperatorSession` true. The switch sets `by=operator`. Under full the acceptance is ruled; under partial the ruling is empty. Correct (`verbs.go:90-121`).
- `Accept`, operator session with a withdrawn test: neither refusal case matches, because the first needs `by==agent` and the second needs `by==caller`. The row is written and the test comes off `Withdrawn`. Correct, and it is the intended override.
- `Accept`, autonomous agent with a withdrawn test: `by=agent`, so the first case refuses. Correct. The operator case sits before the agent case, so there is no path that sets `by` wrongly.
- `Accept`, operator mark set but session closed, status running: `AtAgentSession` is false, the `Running` case refuses, and no `by` is set. Correct.
- `Accept`, blocked status with a stale operator flag: `AtOperatorSession` needs a running status, so it falls to the caller case. Correct.
- `ResetRuns`, `OperatorSessionOpen`, `CloseAgentSession`: `ResetRuns` rewrites the whole record with the mode. `CloseAgentSession` clears only `SessionOpen`, so `OperatorSessionOpen` (`SessionOpen && Operator`) is false once closed. With no record it is false. Correct (`records.go:235-273`).
- `pass`, interrupted entry whose agent is not live: `CloseAgentSession` runs, and the stale `Operator=true` stays but is masked until the next `ResetRuns` overwrites it. Correct.
- `pass`, interrupted entry whose agent is live, then `runAgent`: `prepare` is not called on the resume path (`multillm.go:106-111`). The record keeps the spawn-time mode, as the summary claims. Correct.
- `runAgent` Done check under full: an operator acceptance is ruled, so `Unruled()` is empty and the result is Done. Agent acceptances stay unruled and still halt. The `HaltNamed` ruling pass at `verify.go:142-154` only touches `RulingUnruled` rows. Correct.
- `VerifyTable`, both modes: `Interactive` and the mode and judging text come from one `cfg.VerifyInteractive`. `seatValues` passes `table.Interactive` to `parentdirective.Directive`, which drops the operator-ban line. `seatSpec` sets `Interactive` on the chair only. Correct.
- `LoadConfig` and `ConfigSpec`: `verify_interactive` is `CategoryNone`, a plain bool defaulting to false. Correct.
- `wiring.go` `verbStatus`: a failed `OperatorSessionOpen` returns an error, and the success path fills the field. Correct.

**Findings**
1. LOW, `design`: `verify.go:498-508` (prepare closure) with `shedadapters/multillm.go:93-121`. The recorded mode comes from a second config read (`d.Config()` inside `prepare`), not from the table that is spawned. The table is built earlier by `d.VerifyTable`, which also reads config. If `verify_interactive` flips between the two reads, `runs.json` says `Operator=true` for an autonomous, non-interactive session, or the reverse. A true flag on a session with no operator lets that agent record ruled acceptances that bypass the parent's halt. The window is small. Deriving the mode from the table's `Interactive` would remove it.

I found no other defects in this lens.

**Tool calls:** none. The inherited context already held every code path I needed.
