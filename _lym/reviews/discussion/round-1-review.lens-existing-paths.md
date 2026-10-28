```yaml
lens: existing-paths
outcome: reported
```

Lens existing-paths: tracing every existing code path that writes, reapplies or reads the state each Decision introduces or changes.

**Paths traced, per decision**

- D1/D2, `lym loom decision add` (`internal/discussionparser/append.go:57-66`, which runs `Validate` alone after appending): survives. Added entries carry no id and are skipped. A run whose record predates D1 is not refused.
- D1/D2, darn `checkDarnRecord` (`internal/loomcli/decision.go:339-366`, needs the record committed, unchanged and holding every added entry): survives. The darn gate commits only on a clean check (`gates.go:165-175`), so a D1/D2 finding leaves the record uncommitted. `decision add` stays refused until the writer fixes it, which is the existing contract.
- D1/D2, `ValidateDarnDiscussion` early return (`internal/loomshed/gates.go:139-158`): breaks in a small way. Today the function returns as soon as `Validate` has findings, and runs the history check only on a clean `Validate`. D2 says D1/D2 run "beside" `Validate` in both functions, but does not say where they sit relative to that early return and the history check.
- D1/D2, the loom gate, the Discussion-Burler-Fix gate and the verb (`gates.go:103-133`, `validate.go:66-73`, `entries_gate.go:139-155`): survives. They are one function called through the parity seam.
- D3/D4, gate access to the root and index: survives. `Env.WorktreeRoot` and `Env.PlanIndex` are wired once per run for every row (`internal/loomcli/wiring.go:550,582`). The verb reads the same `c.env` fields (`validate.go:285-289`).
- D3/D4, skipped carry-over blocks and suffixed PATTERN bullets (Go writers: `decision.go`, `identity.go`): survives, because D3/D4 skip both.
- D3/D4, plan amendments (`internal/loomshed/amendment.go`): no Discussion re-gate. The amendment record is plan-side, so there is no path that re-runs the citation check over a settled record after Webster changes the tree.
- D5, guard scan `guardNamesInFile` / `onlyKeepLinesAfter` (`internal/impactset/guards.go:160-185, 194-201`): D5 states the change. Today a `//lym:seam` line between `//lym:guard` and the func trips `GuardMisplaced`. D5 makes the scan admit it, so this survives as stated.
- D5, testtiming keep scan (`cmd/testtiming/keep.go:20-22`): survives without change. Keep may sit above `//lym:` directive lines, so `//lym:seam` stacks below or above keep.
- D6–D9, `PlanContext` builders: survives. There are two builders, `planContextOf` (`entries_gate.go:126-133`) and the verb (`validate.go:271-277`). D9 routes the new data through `PlanContext`, and the parity test pairs the gate with the verb.
- D9, the done-card skip: survives. `ValidatePlan` passes pending cards only (`gates.go:259-269`), and the later cards D6/D8's "later card" coverage reads are pending as well. `ValidateReworkPlan` passes all cards (`gates.go:330`), matching "fresh generation".
- D9, other plan validators: none outside the verb and gates. A grep finds no other `ValidatePlan` or `planContextFindings` caller (`validate.go:289` is the only external caller). Webster's dispatch goes through `ValidateDispatch`, which D9 excludes.
- D7, gate.yaml loading: survives. The wiring already loads `gateCfg` (`wiring.go:406-411`) and the verb tolerates `ErrConfigAbsent` (`validate.go:263-270`), so there is a source for the tier tags in both builders.

**Findings**

1. severity: LOW; class: consistency; location: D2, second paragraph (darn wiring).
   - Summary: D2 puts D1/D2 "beside `discussionparser.Validate`" in `ValidateDarnDiscussion`, but that function returns early on any `Validate` finding and runs the history check only after a clean `Validate`. The record does not say whether D1/D2 findings join `Validate`'s findings before that return, or come after the history check. Depending on the reading, a record with both a malformed heading and a dropped added entry is told one or both defects per re-prompt.
   - Evidence: `internal/loomshed/gates.go:139-158`.

No path traced undoes a Decision's result. The other traced paths survive as the record states.

Tool calls:
1. Grepped for `//testtiming:keep` readers and for `ValidatePlan`/`planContextFindings` callers, because the manifest had neither the keep-scan owner nor the full set of plan-validator callers.
2. Read the keep-directive placement rule in `cmd/testtiming/keep.go`, to check whether D5's stacking survives the second scanner.
3. Grepped for other decision-record readers and writers and for the H3 handling in `internal/discussionparser/identity.go`, to find the paths that write the record besides the writer.
4. Grepped for `CheckAddedDecisionsKept` callers and the amendment flow, to find the darn `decision add` check and whether amendments re-gate the record.
5. Read `checkDarnRecord` in `internal/loomcli/decision.go:337-366`, to check that darn's commit precondition survives D1/D2.
