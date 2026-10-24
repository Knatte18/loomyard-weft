# Fixer report — plan, round 3

Every finding's premise holds against the code at HEAD; every finding is fixed.

## Changes

- **F1 (BLOCKING), fixed as proposed, with one addition per card.**
  - Card 14: the Edit line for the nonexistent `internal/gatecli#runGuardStep` is replaced by `internal/gatecli#guardStepRun` with the proposed signature, and `internal/gatecli#guardPackages` and `internal/gatecli#execution` are added.
    The Intent names `guardStepRun` in place of `runGuardStep`.
    It now says `scope.guards` and `execution.guards` carry `map[int]map[string][]string`, tier to directory to guard names, and that each group is one tier's inner map.
  - Card 20: `commandRefWayForward`, `commandTestConfig`, `commandTestSettings`, `CommandProducer`, battenshed `Config` and `ConfigSpec` are added to Edit.
    The Intent says `commandRefWayForward` loses the word switch and that the helper config drops the key, so the conformance table's Command row still builds.
  - Card 13: `prebuildLyx` and `TestPrebuildLyx` are added to Edit.
    The Intent says the `tier0` default reaches the `go test` arguments only, and that `resolveTags` still returns `""`, so the tier label, the real-git hint, `prebuildLyx` and `redundancyTags` behave as today.
    Addition: so the two Edit entries plan a change, `prebuildLyx` also builds nothing for an explicit `-tags tier0`, and `TestPrebuildLyx` gains a `tier0` row.
- **F2 (MEDIUM), fixed as proposed.**
  Card 15 moves `internal/gatecli/doc.go` from Extends to the end of Edit.
  The Intent says the Counters, record and log section's two sentences change to the D7 rule.
- **F3 (MEDIUM), fixed as proposed.**
  Card 17 moves `internal/shedengine/doc.go` from Extends to the end of Edit.
  The Intent names the four doc sentences that change: Caller labels, `pause_after`, the bounce budget and Field ownership.
- **F4 (MEDIUM), fixed as proposed, except for one row (see the self-check).**
  The card 20 Intent keeps the `TestWire_MarkerAndReportSeamsRefuseAnAbsentTaskWorktree` forced-answer assertion.
  It drops the `Env.Setting` and `seed_ide` assertions planned for `stepCommandRows_FollowTheTasksLocalSwitch`.
  It deletes the `SwitchOffAndOpenTerminalForced` row and its `forced` field.
  It updates the `UnknownKey` row's text and deletes `MissingSwitch` and `NonStringSwitch`.
- **F5 (LOW), fixed as proposed.**
  Card 22's `docs/overview.md` sentence now covers the gate synopsis's `--gate <key>` and the `fabricengine_test` paragraph's `tier1`.
  Card 31's crucible sentence now covers the `//go:build tmux` and `//go:build llm` prose of `crucible/README.md` and `crucible/review-prompt-template.md`.

`lyx loom validate-plan` passes after every change, the self-check changes included.

## Self-check

- fix-claims, MEDIUM: card 20's `stepCommandRows_FollowTheTasksLocalSwitch` cannot keep its off-case steps through `stepCommandRow`, and setting the flag alone forces nothing.
  Disposition: fixed with own change.
  The F4 proposal said only that the test sets `--open-terminal` in place of `Forced`.
  That is wrong on two counts:
  - `stepCommandRow` (`lifecycle_integration_test.go:1192-1203`) fatals unless the row ends Done, and under card 17 a gated-off row ends its step skipped and chains on.
  - The forced answer is built in `battenCLI.wire`, and `c` is wired once in `newCommandRowFixture`.

  Card 20 now creates `plan:internal/battencli#stepGatedOffCommandRow`.
  It seeds a running status at the row with `pause_before` naming the row's `on_done` row, steps once, and asserts that the history ends in a `skipped` entry and the run is paused at that row.
  The forced step re-wires the CLI through `battenCLI.wire` with `openTerminalFlag` set.
- fix-claims, LOW: card 17's Field ownership change would put `entered_row` in a sentence that calls its fields "rewritten on every persist" (`internal/shedengine/doc.go:109`), but resume and halt writes keep it.
  Disposition: fixed as proposed.
  The Intent now gives `entered_row` a sentence of its own, naming the writes that set, clear and keep it.
- fix-proposal, MEDIUM: deleting `ReferenceWithNoDot` drops the only row reaching `commandSettingRef`'s dot check, which `command` keeps (`entries_command_test.go:71`).
  Disposition: fixed as proposed.
  The row now targets `command` with `open_window_command`; this departs from F4's proposal to delete it.
- fix-affected-set, MEDIUM: card 14 misses `internal/gatecli#scopesConfigWith`, whose config seed holds `guard_tier` (`scopes_integration_test.go:39`).
  Disposition: fixed as proposed; added to Edit.
- fix-affected-set, MEDIUM: card 13 misses the `-resources` mode, which takes `resolveTags`' result and branches on `tags != ""` (`resources.go:85`, `168`).
  Disposition: fixed as proposed.
  `runResources` and `measurePackageResources` are added to Edit.
  The Intent has `runResources` hand `tier0` to `listPackages` and `measurePackageResources`, while its tier label and `prebuildLyx` still see `""`.
- fix-consistency, LOW: card 13's ImpactSummary no longer matches its Intent.
  Disposition: fixed as proposed.
  The ImpactSummary now names the resources mode and the prebuild skip.

Every fork returned a report.

## Disputed

None.

## Deferred

None.
