# Card 11 — lyxbin-prebuilt

**Create:**
- `internal/gateslot#PrebuiltLyxEnv`
- `internal/testkit/lyxbin#ErrStalePrebuilt`
- `internal/testkit/lyxbin#prebuilt`
- `internal/testkit/lyxbin#buildOnce`
- `internal/testkit/lyxbin#builds`
- `internal/testkit/lyxbin#buildResult`
- `internal/testkit/lyxbin#buildsMu`

**Edit:**
- `internal/testkit/lyxbin#BuildInto`
- `internal/testkit/lyxbin#Build`
- `internal/testkit/lyxbin#BuildWithLDFlags`
- `internal/testkit/lyxbin/lyxbin_integration_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Prosa:**
- `pattern/PATTERN-testkit.md`
- `pattern/PATTERN-test-speed.md`

**Intent:** D7's kit half: `lyx` is built once and named by an environment variable.
`PrebuiltLyxEnv` is the one declaration of the variable's name, in `gateslot` beside `InheritEnv`, because the gate verb, the harness and the kit all spell it and a kit may not be imported from production code.
`prebuilt` reads it: unset is `("", false, nil)`; set to an existing executable file is `(path, true, nil)`; set to anything else is `ErrStalePrebuilt` wrapped with the path, so a stale value from an earlier build fails loudly instead of running another tree's binary.
`buildOnce` caches one build per test binary and per key, the key being the variable's value joined with the ldflags: an empty-ldflags key returns the prebuilt binary when one is set, else builds once into `os.MkdirTemp("", "lyxbin-")`, which `tmuxkit.Main` has pointed at the package's private directory; a non-empty-ldflags key always builds, once, since a stamped build such as `reedcli`'s switch-seed test can never be the prebuilt binary.
`BuildInto` keeps its signature and returns the prebuilt binary for empty ldflags and a build into `dir` otherwise; `Build` and `BuildWithLDFlags` go through `buildOnce` and no longer build into `tb.TempDir`, so the binary outlives the calling test.
Tests, in the existing integration file with `t.Setenv` and no `t.Parallel`, the environment named as the global state: `TestBuild_ProducesExecutableBinary` becomes a table: unset, two `Build` calls return one path and it is executable; set to a copy of that binary at another path, `Build` returns that path; set to a missing path, `BuildInto` returns `ErrStalePrebuilt` naming it; `TestBuildWithLDFlags_ReachTheBuild` gains a row that the flags reach the build with the variable set.
`PATTERN-testkit.md`'s lyxbin bullet records the variable and the once-per-binary cache; `PATTERN-test-speed.md`'s "every gate run takes a slot" section gains the sentence that a tagged gate run builds `lyx` once and hands it to every package through the variable, with the bound that a forged value misleads only an ungated raw run.

**ImpactSummary:** The three build entry points keep their signatures; every tagged test that calls them now shares one binary per test binary, and no caller changes.
