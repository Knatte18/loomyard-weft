# Card 8 — testmain-internal-m-z

**Create:**
- `internal/mergeresolve/testmain_test.go`
- `internal/modelspec/testmain_test.go`
- `internal/orchengine/testmain_test.go`
- `internal/output/testmain_test.go`
- `internal/parentdirective/testmain_test.go`
- `internal/parentreview/testmain_test.go`
- `internal/pattern/testmain_test.go`
- `internal/planparser/testmain_test.go`
- `internal/proc/testmain_test.go`
- `internal/quarrycli/testmain_test.go`
- `internal/reedengine/render/testmain_test.go`
- `internal/segmentcolor/testmain_test.go`
- `internal/selfreportcli/testmain_test.go`
- `internal/selfreportengine/testmain_test.go`
- `internal/shedadapters/testmain_test.go`
- `internal/shedbuild/testmain_test.go`
- `internal/shedcheck/testmain_test.go`
- `internal/shedengine/testmain_test.go`
- `internal/shedrecipe/testmain_test.go`
- `internal/shedrun/testmain_test.go`
- `internal/shedtransient/testmain_test.go`
- `internal/shell/testmain_test.go`
- `internal/shuttleengine/testmain_test.go`
- `internal/state/testmain_test.go`
- `internal/statuscommit/testmain_test.go`
- `internal/stencil/testmain_test.go`
- `internal/stencilstore/testmain_test.go`
- `internal/summaryparser/testmain_test.go`
- `internal/termwindow/testmain_test.go`
- `internal/twilldirective/testmain_test.go`
- `internal/vscode/testmain_test.go`
- `internal/weftname/testmain_test.go`
- `internal/yamlengine/testmain_test.go`

**Uses:**
- `internal/testkit/tmuxkit#Main`

**Intent:**
The same change as card 7 for these test packages, none of which declares a `TestMain` today: an untagged `testmain_test.go` whose `TestMain` is `os.Exit(tmuxkit.Main(m))`, with no build constraint and a one-line comment saying why it exists (D4).
It uses the package clause the package's in-package test files use, and in a package whose test files are all external that external clause, except in `internal/proc`, where it is the external `proc_test`: `tmuxkit` imports `proc`, so an in-package test importing `tmuxkit` would be an import cycle.
None of these packages spawns git from its tests, so none calls `gitkit.HermeticGitEnv`.
The existing tests of each package run unchanged under the new entry point.
