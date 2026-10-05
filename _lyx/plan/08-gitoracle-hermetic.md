# Card 8 — Run gitoracle's tests under the hermetic git environment

**Create:**
- `internal/gitrepo/internal/gitoracle/testmain_test.go`

**Intent:**
`internal/gitrepo/internal/gitoracle` gains a `TestMain` that calls `gitkit.HermeticGitEnv()` before `m.Run()`, the shape every other git-touching package uses (D5, `PATTERN-hermetic-git-tests`).
The package's production code runs `gitexec.RunGit`, so any test in it that reaches a read spawns git, and the TestMain keeps that git from inheriting the operator's global config.
The leaf rule in `leaf_enforcement_test.go` scans production imports only, so importing `gitkit` from a test file is allowed.
The package has no tagged test file, so `tmuxkit.Main` is not required; the TestMain stays untagged and spawns nothing itself.
Its comment cites `PATTERN-hermetic-git-tests` as its sibling `testmain_test.go` files do; card 10 renames that reference with the rest.
Check `cmd/lyx/hermeticenv_test.go`'s allowlist for a gitoracle entry and remove it if one exists, since the package now meets the rule outright.
