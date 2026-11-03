```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the diff (seatengine CheckMarkers/seatLabel/readStencilSet, preflightshed checks, wireDarn).

**Per-function list**

- `Table.seatLabel` (table.go:218): lead with `Name=""`, prefix `darn` gives `darn`. Advisor `critic` gives `multi-critic`. Correct.
- `Table.CheckMarkers` (prompt.go:283), a stencil with `{{.subagent_model}}` and no value: `TopLevelMarkers` lists it, `fills` is false, and the error names the seat, the stencil, the marker, the directory and the way forward. Correct.
- `CheckMarkers`, a value present but empty or whitespace-only and not optional: `fills` only checks key presence (prompt.go:311-315). `FillWith` treats an empty or whitespace value as unfilled (`unfilledTopLevelMarkers`, stencil.go:298). So `CheckMarkers` passes where the real fill fails. Wrong, but narrow. The darn table cannot reach it today, because `DarnTable` refuses empty required fields and the empty values it can carry are all in `Optional`. It does break the claim that "a table that passes composes no prompt with an unfilled marker". Finding F1.
- `CheckMarkers`, a marker in an `{{if}}` branch that is present but empty: `FillWith` has `presentButEmptyBranchMarkers`; `CheckMarkers` has no equivalent. The doc comment says "top-level", so this is consistent and not a defect.
- `CheckMarkers`, blocks: `append([][]byte{body}, slices.Collect(maps.Values(blocks))...)` visits every block. `unfilled` is deduplicated and sorted, so map order does not matter. Correct.
- `CheckMarkers`, failures: an unreadable stencil comes back from `readStencilSet` already naming the seat label. A parse error is wrapped with the label and stencil name. Correct.
- `fills` with a nil `Values` map: reading a nil map is safe. Correct.
- `readStencilSet` (table.go:359): a read failure of the stencil or a block, and an `IncludeNames` failure, each carry the label. Correct.
- `validateStencil`: it now iterates the sorted block names. Its error order is deterministic and no worse than before. Correct.
- `validateOutputs`: `owners[output] = label` now stores the role, so the "also seat X's output" message names the role. Correct.
- `validateValues(fmt.Sprintf("seat %q", label), ...)`: the message for an empty lead value reads `seat "multi": value ...`. Correct.
- `composeSeatPrompt`: the signature changed, and the callers `startSeat` and `Prompts` are updated. A fill failure names the role. Correct.
- Error labels in `engine.go:303`, `resume.go:72`, `strandNames`, `seatValues`: all use `seatLabel`. Correct. The three name-validation errors (table.go:275, 280, 283) keep the raw name on purpose.
- `preflightProducer.Call` (preflight.go:182): the loop runs the batcher check, then the stencils check. The first non-empty reason returns Stuck. Cancellation inside either check returns its error. A nil check function returns "". Correct.
- `stencilsFailure`: on a check error it checks cancellation first, then logs a Warn and returns the reason prefix plus the error text. Correct. The error text already contains "way forward:", so the reason carries it.
- `wireDarn` `PreflightStencils` (darnwiring.go:261): it calls `darnTable(DarnTold{})`. `PriorWork=""` becomes `orNone`, so `prior_work` is filled as in a real spawn. It then runs `withProjectDirective`, as the real path does, so `project_directive` is in the values. `table.Gate` is not set, but it does not affect markers. The stencils directory is the same `stencilsDir` the real run reads. Correct.
- `wireDarn`, `darnTable` failing (for example an empty `SubagentModel` or an unresolvable writer): the error returns unwrapped as the Stuck reason with the prefix "seat stencils cannot be filled: ". The prefix misdescribes a config or table-build fault, and the message has no way forward. The real run would also fail on that error. Minor misreport. Finding F2, LOW.
- `preflightEntry` (entries_simple.go:25): a nil `PreflightStencils` gives a nil check, so no check runs. Correct.

**Findings**

- F1, LOW, design, prompt.go:311-315: `fills` treats a present-but-empty or whitespace-only, non-optional value as filling the marker, which `FillWith` refuses (stencil.go:298). `CheckMarkers` can therefore pass a table whose real fill fails. Evidence: prompt.go:310-315, stencil.go:292-303. Fix: `fills` should require `strings.TrimSpace(value) != ""` for table and seat values, unless the marker is in `Optional`.
- F2, LOW, design, darnwiring.go:261-267 with preflight.go:148: a failure to build the table (config, model-spec, project-record read) is shown under "seat stencils cannot be filled:" and carries no way forward. Evidence: darnwiring.go:262-265, preflight.go:148.

**Tool calls:** none. The inherited context already held everything needed.
