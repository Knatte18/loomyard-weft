# Round 1 fixer report

## Changed

- F1: `TestWatcher_SkillSkipCauses` selects its final assertion with a `reloaded` bool instead of a bare string.
- F2: `TestStartup_SkillSkipCauses` fails on an undeclared `elapsed` expectation through a `default` case.
- F3: `TestRunner_LoadSkillAndSkillUnknown_GuardStrands`, `TestWatcher_SkillSkipCauses` and `TestStartup_SkillSkipCauses` call `t.Parallel`, subtests included.
  `go test -race -count=5` over them passes.

## Deferred

None.
