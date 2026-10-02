---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    location: internal/loomcli/parity_test.go:546
    summary: four new doc comments break mid-sentence at a comma that joins a list item, a compound predicate or a subordinate phrase, not at an independent-clause boundary
---

The review range resolved from `_lyx/shed/rework-generations/status.json` (`product.parent: main`), merge base `9ae5f9405`, covering every commit through `21a1a9783`.
`go build ./...` and `go vet` pass, the `loomshed` and `loomcli` unit tests pass on HEAD, and no file in the range is reported by `gofmt -l`.

### [LOW] Four doc comments break inside a sentence at a non-clause comma

**Location:** `internal/loomcli/parity_test.go:546`–`547`, `internal/loomcli/wiring_commitstatus_test.go:411`–`412`, `internal/loomcli/wiring_rework_integration_test.go:217`–`218`, `internal/loomshed/gates_test.go:437`–`438`.

**Issue:** CONSTRAINTS.md's Comment Line-Break Convention defers to `scribe:prose`, which allows a break inside a sentence only at an independent-clause boundary and says a comma joining a list item or a compound predicate does not trigger one.
Each of these comments breaks a single sentence across two lines at such a comma:

- `parity_test.go:546` breaks the list of gate cases after `(stuck),`, between two list items.
- `wiring_commitstatus_test.go:411` breaks before `whether the highest round is unclassed or classed but uncommitted`, a subordinate phrase with no subject of its own.
- `wiring_rework_integration_test.go:217` breaks after `over a real fabric pair,`, separating the subject `Env.SkipPlanReview` from its verb `answers`.
- `gates_test.go:437` breaks between the compound predicates `writes ...` and `declares ...`, inside one sentence.

**Fix:** join each sentence onto one line, touching no other line.

## Focus verdicts

- **The four named comments.**
  None holds one sentence per line under the rule; all four are F1.
- **Any other new or changed Go comment in the range.**
  Every adjacent pair of added comment lines in the range whose first line does not end a sentence was checked.
  The remaining mid-sentence breaks all precede an independent clause and comply: `internal/loomcli/wiring.go:94` (`so the moved-from deletions ... land`), `internal/loomcli/wiring_rework_integration_test.go:70`, `internal/loomengine/rework.go:75`, `internal/loomshed/generation.go:48` and `:84`, `internal/loomshed/prrework.go:5`, `internal/planparser/validate_test.go:1902`, `internal/shedrecipe/entries_bouncer.go:122` and `internal/websterengine/runlevel_test.go:2271`.
  The colon and semicolon breaks at `wiring.go:66` and `:96`, `generation.go:78`, `prrework.go:4`, `prrework_test.go:451`, `shedadapters/bouncer.go:88`–`89`, `websterengine/archive.go:93` and `runlevel_test.go:2270` are each followed by an independent clause and comply.
  Not a defect beyond F1.
- **Round-1 fix commits as code.**
  The `roundFor`/`rekeyRound`/`commitRound` rework, the atomic `writeRoundRecord`, the working-tree check in `PlanReviewSkippable`, the `ArchivedWebsterDirs` error path and the `<empty>` first_card placeholder were read as code; no defect found.
  A crash between the temp write and the rename in `writeRoundRecord` leaves `record.json.tmp`, but every path that follows rewrites and renames it before the round commit, so it never lands.
