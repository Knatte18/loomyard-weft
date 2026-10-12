# Card 7 — testmain-internal-a-l

**Create:**
- `internal/agentname/testmain_test.go`
- `internal/batcher/testmain_test.go`
- `internal/battenrecipe/testmain_test.go`
- `internal/battenshed/testmain_test.go`
- `internal/buildinfo/testmain_test.go`
- `internal/buildvcs/testmain_test.go`
- `internal/clihelp/testmain_test.go`
- `internal/configengine/testmain_test.go`
- `internal/configreg/testmain_test.go`
- `internal/configsync/testmain_test.go`
- `internal/darnengine/testmain_test.go`
- `internal/discussionparser/testmain_test.go`
- `internal/dotgit/testmain_test.go`
- `internal/editdirective/testmain_test.go`
- `internal/envsource/testmain_test.go`
- `internal/friction/testmain_test.go`
- `internal/frictionengine/testmain_test.go`
- `internal/fslink/testmain_test.go`
- `internal/fswatch/testmain_test.go`
- `internal/fsx/testmain_test.go`
- `internal/githubclient/testmain_test.go`
- `internal/helpcli/testmain_test.go`
- `internal/lock/testmain_test.go`
- `internal/loggerconfig/testmain_test.go`
- `internal/loomengine/testmain_test.go`
- `internal/loomrecipe/testmain_test.go`

**Uses:**
- `internal/testkit/tmuxkit#Main`

**Intent:**
Give each of these test packages, none of which declares a `TestMain` today, an untagged `testmain_test.go` whose `TestMain` is `os.Exit(tmuxkit.Main(m))`, so its `t.TempDir` trees and any temp file its tests create live in the process run directory and go with it (D4).
The file carries no build constraint, so it compiles under every tag set and platform the package's other test files do, and a one-line comment says why it exists.
It uses the package clause the package's in-package test files use, and in a package whose test files are all external that external clause, except in `internal/agentname` and `internal/lock`, where it is the external `<package>_test`: `tmuxkit` imports `agentname` directly and `lock` through `lyxtemp`, so an in-package test importing `tmuxkit` would be an import cycle.
None of these packages spawns git from its tests, so none calls `gitkit.HermeticGitEnv`, and in an untagged run `tmuxkit.Main` spawns nothing.
The existing tests of each package run unchanged under the new entry point; card 10's scan then requires the entry point for every test package.
