```yaml
lens: fix-way-forward
outcome: reported
```

Task: check that the ways forward this fix introduced can be carried out from the state they name.

**Way-forward list:**
1. D3: a reach turn's failing result goes to the writer, who adds tests and commits or ends the turn unchanged. This can be carried out.
2. D3: the reach-turn count of failing results since the last pass counts shuttle arrivals, not inner gate runs. This one is a finding, below.
3. D10: the step is built in a later darn that the parent opens from the D11 board note's decision. This can be reached, because D11 files that note.
4. D11: the board note asks the parent whether to build D10 and list `mutation`. This can be carried out once the trial gives its counts.
5. D12: accepting `mutation` and refusing `mutation` without `reach` move to D10's later build. Both can be reached, through item 3.
6. D10 criteria are "checked at D10's later build". These can be reached, through item 3.
7. D3 criterion: with `darn.gate_fixer` set, a reach turn reaches the Darn-Write writer. This can be tested.

**Findings:**

- **MEDIUM**: the new D3 bullet says "the count of failing results since the last pass counts shuttle arrivals, not inner gate runs".
  - **Summary:** the record requires the gate to count arrivals but gives the inner gate no way to tell an arrival from the wrap's re-run, so an implementer cannot carry out the rule as written.
  - **Evidence:** `internal/gatefixer/rowgate.go:153-164` and `:218` call the same `inner()` closure with no argument, both for the first judgment and for the re-run after the fixer.
  - **Evidence:** the `shuttleengine.Gate` signature is `func() (GateResult, error)`, so the inner closure gets no arrival marker.
  - **Evidence:** the bullet names neither a wrap-side count nor a flag passed to the inner gate. The plan writer would have to invent one, such as the wrap counting arrivals or the gate keying its count on something the wrap records.
  - **Fix needed:** the record should say which side keeps the count and from what signal.

Further tool calls: none.
