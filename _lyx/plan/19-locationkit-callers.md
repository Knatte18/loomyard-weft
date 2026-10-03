# Card 19 — locationkit-callers

**Edit:**
- `//cmd/lyx/constructoranchoring_test.go`
- `//cmd/lyx/notransients_test.go`
- `fabricengine/portallauncher_test.go`
- `fabricengine/hubworkspace_test.go`
- `fabricengine/launcher_content_test.go`
- `fabricengine/origin_test.go`

**Uses:**
- `testkit/locationkit/locationkit.go`

**Intent:**
Move the three private synthetic `*lyxcwd.Location` builders onto card 18's `locationkit` (#325 §6).

`anchoringFixture` (`cmd/lyx/constructoranchoring_test.go`), `notransientsFixture` (`cmd/lyx/notransients_test.go`) and `newPortalLauncherTestLocation` (`fabricengine/portallauncher_test.go`) go, and their callers call `locationkit.Location`.
`newPortalLauncherTestLocation` is also called from `hubworkspace_test.go`, `launcher_content_test.go` and `origin_test.go` in the same package; every one of those calls converts in this card, so fabricengine's tests still compile after it.
`newPortalLauncherTestLocation` derived the worktree name with `filepath.Base(worktreeRoot)`, which its callers now do inline, or pass the name directly where they built the root from it.
`origin_test.go`'s header comment, which says its Locations are built as `newPortalLauncherTestLocation` does, names `locationkit.Location` instead.
Apply D10 to every touched test.

**ImpactSummary:** Six untagged test files, two in cmd/lyx and four in fabricengine, drop their private builders or the calls to them for one kit call; no assertion changes.
