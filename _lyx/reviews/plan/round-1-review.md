---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: _lyx/plan/00-overview.md (## verify:)
    summary: verify never runs cmd/lyx, whose untagged tests scan both card 3 prose files for denied commands and check the Long form of the intake list and intake import commands cards 1 and 2 rewrite
    fix: In _lyx/plan/00-overview.md's verify block, change the untagged line `go test ./internal/boardcli/... ./contracts/...` to `go test ./internal/boardcli/... ./contracts/... ./cmd/lyx` and leave every other line and the order unchanged; validate-plan --overlay with this change reports ok, as the baseline does.
---

### [LOW] verify leaves out the cmd/lyx tests that check this plan's prose and help text

**Location:** `_lyx/plan/00-overview.md`, `## verify:`

**Issue:** The decision record's `PATTERN-no-denied-recovery` constraint covers both files card 3 edits.
The test that enforces it is `TestNoDeniedRecovery_ScannedSurfaceHasNone` in `cmd/lyx/norecoverydeny_test.go`, which scans `contracts/stencils` and `plugins/ly/skills` in the untagged tier.
`cmd/lyx/clitree_test.go`'s `TestCLIForm_EveryCommand` checks the `Long` form and its `maxLongBytes` cap on every command, including the `intake list` and `intake import` `Long` texts that cards 1 and 2 rewrite.
The verify section runs only `./internal/boardcli/...` and `./contracts/...`, so neither check runs at the plan's integration step.
Read narrowly, the rubric's verify bullet does not cover `cmd/lyx`, since no card targets it.
The finding is LOW because the landing gates run `./...` and would still catch a break.

## Rejected

- Fidelity (focus 1): every D1–D5 step is carried.
  D1 and the `doc.go` part of Scope are in card 1, D2 in card 2, and D3, D4 and D5 in card 3.
  The four Constraints are in card 1 (`PATTERN-cli-cobra`: `--full` is a flag on the existing `list` command) and card 3 (`PATTERN-role-skills-typed`, `PATTERN-no-denied-recovery`, `PATTERN-test-economy`).
  Card 3's stencil bullets track D3's and D4's bullets one for one: the wake check, the empty and error cases, the one-fork rule, the orch-note rule, the three outcomes, the run-held fold, the single close re-run, the upsert bound, the return line and `failed` handling.
  None adds scope.
  Card 3's SKILL.md line saying `import` prints the short result is licensed by Scope's "matches the new output and flag".
  No card edits the task's board entry.
- Card 1 refusal rows (focus 2): each row reaches a distinct branch.
  The list-failure row's `failList` 500 fails `selfreportengine.ListOpenIssues`, which `intakeList` calls first.
  The board-load-failure row lets the list succeed and then writes `{` to board.json, so `Board.RecordedIssues` → `loadStore` → `state.ReadJSON` returns `ErrDecode` (`internal/state/state.go:110-115`).
  `failList` is a test-fake field read only by `fakeInbox.list`, and the card says so.
- Card order and glyphs (focus 3): card 2 uses nothing card 1 creates or changes.
  `intakeImport` keeps its signature, so it takes no arrow.
  `intakeList` carries the re-sign arrow for its new `full` parameter.
  `intakeCommand`, `intakeIssueView`, `fakeInbox`, a struct, and `fakeInbox.list` keep their signatures and correctly take no arrow.
- Attack surface (focus 4): card 1's Intent bounds `--full` (read-only, same filtered set, no other issue, no write).
  Card 3's Intent bounds the fork's upsert to the labels of the entry it just folded into, keeping every existing label.
  The fold that upsert follows is refused for a run-held entry (`TestIntakeImport_RefusedBeforeAnyWrite`'s "into a run-held entry" row).
- Verify order (focus 5): the order is build, vet integration, vet tmux, vet llm, untagged, integration, as the plan template states; `./contracts/...` covers the stencil contract tests in `contracts/stencils`.
  The one gap is F1.
- Card 2 Rename (focus 6): the test is a real symbol rename.
  The Old side is a glyph and the New side a `plan:` handle, and both project into the card's targets, so the assertion changes the Intent describes fall inside the card.
  `TestIntakeImport_RecordedIssueIsNoOp` under `Uses:` is a read-only reference the card relies on for D2's recorded-no-op clause; no card targets it, so it creates no dependency edge.
  It is not a defect.
- Card 2's fold test expecting `labels` `[bug undecided]` matches `foldIssue` (`internal/boardengine/intake.go`), which carries every known issue label onto the target.
