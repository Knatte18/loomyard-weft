```yaml
lens: code-affected-set
outcome: reported
```

**Lens code-affected-set: per change, the set found and whether the diff covers it**

- **`ResetRuns(p, head, operator)` signature.** Production caller `verify.go:507`, plus the tests in `records_test`, `verify_test` and `driftcli/capture_integration_test.go:221`. All pass three arguments. Covered.
- **`VerbStatus.OperatorSession`, `AtOperatorSession` and `OperatorSessionOpen`.** The only non-test builder of `VerbStatus` is `driftcli/wiring.go:73`, which sets the field. `Accept` is the only consumer. `spentRefusal` in `cli.go` uses `AtAgentSession`, which is unchanged. Covered.
- **New key `verify_interactive`.** Set: `template.yaml`, `config.go` (field, struct tag, `Categories` as `CategoryNone`, `LoadConfig`), `doc.go`, `config_test` and `configreg_test:80`. A grep for `verify_max_runs` and `baseline_model` finds no other fixture or doc that enumerates the drift keys. `docs/overview.md` names no drift key list. Covered.
- **Stencil `drift-template-verify.md` and the new markers `mode_rules` and `judging_rules`.** `markers_test` is updated, and the stencil's header comment is updated. The autonomous constants render the original two paragraphs, apart from the dropped "one autonomous session" wording. Covered.
- **`Accept` and `Withdraw` help text and `doc.go` text.** Updated in `cli.go` and `doc.go`. Covered.
- **Seat table `Interactive`.** The seat engine drops the operator-ban line and sets `Interactive` on the spec through `seatValues` and `seatSpec`, so no seatengine change is forced. Covered.
- **File header of `internal/driftengine/seats.go`, line 3.** The comment still reads "Each agent is one non-interactive lead seat". The diff leaves it wrong for the verify seat. Not covered.
- **`pattern/PATTERN-parent-directive.md`, bullet 1.** It says the discussion role's interactive questions are "the one place a question reaches a person", coming from the `{{.mode_rules}}` marker. The diff adds a second such place: Drift-Verify's `mode_rules` text now has the agent ask the operator in the pane. The entry is not updated. Not covered.

**Findings**

1. **LOW, consistency.** `internal/driftengine/seats.go:3`. The file header says "Each agent is one non-interactive lead seat whose one output is its report". That is now false for the verify seat under `verify_interactive`. `doc.go` was updated for the same claim, and `seats.go` was missed. Evidence: `seats.go:3`, against `seats.go:238` (`Interactive: cfg.VerifyInteractive`) and `doc.go`'s "The verify seat is interactive under `verify_interactive`".

2. **MEDIUM, consistency.** `pattern/PATTERN-parent-directive.md:5`. The bullet names the discussion role as the one place a question reaches a person, through its `{{.mode_rules}}` marker. The diff makes the Drift-Verify seat a second place: `operatorVerifyMode` at `seats.go:52-53` is filled into `{{.mode_rules}}`, and the parent directive then drops its operator-ban line. The PATTERN rule is unchanged and unamended. The repository's own rules say a rule written in a rule document outranks a convention inferred from code. The rubric's "Declared PATTERN changes landed" check does not apply, because the directive declares no entry here. This is a stale entry the diff makes wrong. Evidence: `PATTERN-parent-directive.md:5`, `seats.go:50-53`, `parentdirective.go:36-43`.

Not findings:
- The ask-the-operator phrase scan in `stencils.go` reads stencil files, and the drift stencil now holds only markers. The Go constants are outside it, so it still passes.
- The `AskUserQuestion` ban sits in the Go constant and not in a stencil, which matches the Discussion chair's `modeRules` pattern.
- No `Pattern-Departure:` trailer is present, and none is needed for the points above.

**Tool calls**
1. `grep` for `AskUserQuestion`, drift docs and "non-interactive": my context lacked the other stencils that carry the ban, the docs that name drift keys, and stale "non-interactive" comments.
2. `grep` for `VerbStatus{` builders and seatengine interactive tests: my context lacked a repo-wide list of other `VerbStatus` construction sites and the tests that pin the chair spec.
