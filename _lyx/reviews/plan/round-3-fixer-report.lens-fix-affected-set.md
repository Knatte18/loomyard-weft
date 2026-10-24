```yaml
lens: fix-affected-set
outcome: reported
```

Checked the set-premise findings by rerunning their searches: F1 card 14, F1 card 20, F1 card 13, F4 and F5. F2 and F3 rest on single docs, not sets, so they were out of this lens. I left no finding unchecked.

**MEDIUM: card 14 still misses one `guard_tier` consumer.**
- Changed text: card 14's Edit list (the F1 fix) and its Intent, which says the gatecli test helpers' config seeds drop `guard_tier`.
- Evidence: the `guard_tier` hit at `internal/gatecli/scopes_integration_test.go:39` sits inside `scopesConfigWith` (line 34). The card lists only `scopesConfig` (line 27).
- Cause: the review called card 14's search accurate, so the fix added only `guardStepRun`, `guardPackages` and `execution`.
- Fix: add `internal/gatecli#scopesConfigWith` to the Edit list.
- Every other hit falls inside a listed symbol: `runTest`, `execution.runGuard`, `computeScope`, `scanGuards`, `tierRun`, `planTierRuns` (including the `tiers.go:58` doc line), `planConfig`, `TestPlanTierRuns`, `TestGateTest_Scopes`, `scenarioConfig`, the gateslot `Config`, `configFile`, `ConfigSpec` and `decodeConfig`, and `TestTargetsFor`. `loomshed:118` is the raw config string the card's `roundGateConfig` names.

**MEDIUM: card 13 misses the `-resources` mode, which shares the tag path.**
- Changed text: card 13's new Intent sentence, "The `tier0` default reaches the `go test` arguments only: `resolveTags` still returns `""`…".
- Evidence: `runResources` (`cmd/testtiming/resources.go:66`) and `measurePackageResources` (`resources.go:166`) also take `resolveTags`' result. They branch on `tags != ""` for the `-tags` argument (`resources.go:168`) and the tier label (`resources.go:85`), and call `prebuildLyx` (`resources.go:78`). The tool's usage comment (`main.go:26`) says "-tags and -full pick the tier as in the timing mode".
- Effect: the card puts the `tier0` default into `run` only, and neither resources function is on the card. After the retag, an untagged `-resources` run would measure packages whose untagged files hold no tests.
- Fix: add `cmd/testtiming#runResources` and `cmd/testtiming#measurePackageResources` to the Edit list, and say in the Intent that the resources mode's default also passes `-tags tier0` to `go test`.

**Card 20, no finding.** Every switch-key and `Forced` hit sits on the card after the fix: `commandEntry`, the two way-forward constants, `commandTestConfig`, `commandTestSettings`, `TestCommandEntry`, `CommandRow`, `CommandDeps`, `CommandProducer` and `.Call`, `TestCommandProducer`, the battenshed `Config` and `ConfigSpec`, `shedadapters/doc.go`, the wire seams test and `stepCommandRows`. The file comments of `entries_command.go:1`, `command.go:1` and `command_test.go:1` also name the switch. They sit outside any glyph, and the card's prose covers rewording them only implicitly. I'm not raising that.

**F4 and F5, no finding.** The F4 rows the fix names (`UnknownKey`, `MissingSwitch`, `NonStringSwitch`, `ReferenceWithNoDot`, `SwitchOffAndOpenTerminalForced`) match the tables. F5's crucible `//go:build` sites (`README.md:199` and `review-prompt-template.md:122`, `129`, `132`, `203`) are covered by card 31's new sentence, which names both files.

Tool calls:
1. Grepped card 14's identifiers across the module, because the context lacked a module-wide hit list.
2. Grepped `switch` in the card 20 packages, because the context lacked a full hit list beyond the review's examples.
3. Grepped `tags` and `prebuildLyx` in `cmd/testtiming`, because the context lacked the full set of tag consumers.
4. Mapped each card 14 hit to its enclosing symbol, because I needed the symbols to compare against the Edit list.
5. Read `runResources`' tag handling, because the third call showed it calls `prebuildLyx` and the context lacked its body.
6. Read the `loomshed:118` line, the `tiers.go:58` comment and the `scopesConfig*` and `roundGateConfig` declarations, because I needed to place the hits outside listed symbols.
