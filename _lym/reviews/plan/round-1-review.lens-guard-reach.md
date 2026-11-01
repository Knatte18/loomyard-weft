```yaml
lens: guard-reach
outcome: reported
```

Task: guard-reach lens. For each guard, refusal or branch a card adds, I traced whether a real entry point can reach it.

**Per guard: reachable or not**

- **Card 1**
  - `LoadTapestryConfig` refusals for model, depth, `parts_max` and chair timeout: reachable through hub or profile values.
  - `PartsChairTimeout` error: reachable only through a `plan_timeout_min` of 0, which the card itself says.
- **Card 3**
  - `Parse` errors and every `Check` row: reachable from a written parts file. `part-name-archive` is reachable because `ValidateSeatName` is `agentname.ValidateRole`, a dashed grammar that admits `archive-x` (`internal/seatengine/table.go:56-60`, `internal/agentname/agentname.go:101`).
  - `Validate` absent record: a raw I/O error, which is exempt.
- **Card 4**
  - `part-spot-target`: reachable when a card targets an existing shared-spot path under `Edit`.
  - Part-scope exemptions in `handle-dangling` and `path-missing`: reachable.
- **Card 5**
  - `resolvePass` exemption for `Part.Paths`: reachable.
  - `planGatePass` dropping `caller-uncovered`: reachable.
- **Card 6**
  - `language:`/`root:` mismatch and the cross-part duplicate create: reachable, since nothing upstream checks across parts.
  - Unmet-export refusal: **not reachable from production** (finding F1).
  - Read or write failure: I/O, exempt.
- **Card 7**
  - `ValidatePart` absent parts file: reachable through `validate-plan --part` on a run that did not split.
  - `ValidatePart` malformed parts file: reachable, because the chair edits `parts.md` when it respells an export (card 14, chair Step 1).
  - `ErrPartUnknown`: reachable through a CLI name.
  - Part directory parse error: reachable.
  - `part-export-missing`: reachable.
  - `JoinParts` missing part overview: reachable, for example when a writer stopped before writing it and the chair joins.
  - `JoinParts` failing part: reachable when a part was edited after its gate.
  - Clearing `pattern-line-unprescribed` with `DecisionRecordPath`: that check runs only with the path set (`internal/loomshed/patternplan.go:38`), so the exclusion holds.
- **Card 9**
  - Writer on the lead and the mixed table: no production builder sets either, so these are fail-loud wiring guards. Test rows reach them.
  - Writer failed start ends the step: reachable from the `Engine.Run` start loop (`engine.go:250` site).
  - Done line in `watch`: reachable.
  - Every `seat.Required` site is re-pointed to `endsStep`: `watch` (`engine.go:130`), the `Run` start failure (`engine.go:250`) and `QuietHold` (`prompt.go:171`).
  - The lead-directive check (`table.go` `Validate`) and the Accept rule (`validateSeatRoles`) are both changed by this card.
- **Card 10**
  - Resume: a found writer gets no up line; a writer not found that finished, or that lacks outputs or fails Accept, is handled. Reachable, and it covers the other `seat.Required` site in `resume.go`.
  - Hold deferral: today `outstanding` returns true for any unended run whose up line was not sent (`work.go`). The card sets the up-line test aside for writers, so a writer is deferred only while mid-turn. Reachable.
  - `join` marking a writer when the chair's end arrives first or together: reachable on both select cases (`engine.go:345-378`).
- **Card 11**
  - Every `writerExit` branch is reachable: start error, died or wait error, timeout, failed gate, Accept error, and lost at resume with outputs missing. `SeatResult` already carries `Result`, `AcceptErr` and `WaitErr` (`engine.go:30-41`).
  - Unrecognized outcome: defensive, as the chair's own branch is.
- **Cards 13 and 14**
  - Unreadable fragment: reachable.
  - Bad depth and `gates(part)` error: declared wiring guards.
  - Absent or malformed parts file: reachable. The malformed case comes from a chair edit, and since the table source runs before `Probe` (`multillm.go` `Call`), the live-strand way forward applies.
  - Chair timeout error: reachable with a 0 `plan_timeout_min`.
- **Card 15**
  - Failing `archiveWebster` and the retry: reachable.
  - Absent directory no-op: reachable.
- **Card 16**
  - `parts` gate requiring `PartsMax` above 0: `loomcli` fills it from config, which already refuses values below 1, so it is a wiring guard.
  - `part_gates` with a name other than `plan-part`: reachable from recipe config. Test rows reach both.
- **Card 18**
  - `validate-parts` off-recipe refusal: reachable on darn. `refuseVerbOffRecipe` runs in `armAt` for every loom verb (`arm.go:257`).
  - `--part` flag exclusivity: reachable.
  - Config error envelope: reachable.
  - The claim that only tapestry holds a `PlanWrite` row holds; grep finds only `contracts/recipes/tapestry-recipe.yaml`.
- **Card 19**
  - Off-recipe refusal: reachable.
  - "Not running at the row": reachable. Goto refuses a running run and leaves it halted (`shedengine/goto.go:106`), and the driver sets `running` with the current producer while a row steps (`shedengine/run.go:310-311`).
  - Non-chair caller: reachable.
  - Chair name formed with `agentname.Format(shortname, slug, SeatRole(...))`: matches `strandNames` (`prompt.go:25-35`), and `checkName` guarantees that the chair holds exactly that name.
- **Card 20**
  - Latest-Skipped step: reachable in a split run.
  - Both writers latest-Skipped: reachable only through a flag flipped between the two row entries. That is contrived but possible.
  - History read error: I/O, exempt.

**Findings**

F1 — LOW — design — card 6 (`planparser.JoinParts`, unmet-export refusal), as reached through cards 7 and 19
Summary: the "export not created, re-signed or deleted" refusal cannot be reached from `lym loom join-parts`.
- Card 7's `loomshed.JoinParts` runs `ValidatePart` on each part first. It refuses the first part with a blocking finding, and `ValidatePart` already reports `part-export-missing` using "the same test `planparser.JoinParts` applies".
- So the input that would reach the `planparser` refusal is always rejected upstream, by `ValidatePart`'s `part-export-missing`. Only card 6's unit test reaches it, and card 19's "unmet export passed through" row uses a fake `join`.
- Evidence: `_lym/plan/07-loomshed-part-checks.md:46` (part-export-missing, same test as JoinParts) and `:58` (ValidatePart on each part before the join); `_lym/plan/06-planparser-join.md:355` (planparser refusal); `_lym/plan/19-join-parts-verb.md:476,482`.
- Fix: keep the `planparser` refusal as the format's own contract, but have card 7's or card 19's Intent name `ValidatePart`'s `part-export-missing` as the path that reaches it in production. Alternatively, drop the duplicate test from one of the two places.

Tool calls:
1. `SeatResult` fields and `ValidateSeatName`: to check that card 11's branches have the fields they read, and that the archive-prefix check is reachable. The manifest did not include them.
2. The `refuseVerbOffRecipe` caller, run states and `strandNames`: to trace the reach of the card 18 and 19 recipe refusals, the status guard, and chair-name formation.
3. `armAt`, the role grammar and the goto state checks: to confirm the refusal ordering and the halted-at-row state.
4. How shuttle handles a gate closure error: no match, so it contributed no finding.
5. The checks `planContextFindings` runs: to see whether the part-scope exemptions recur in context checks.
6. Where `pattern-line-unprescribed` runs: to check card 7's exclusion of it.
