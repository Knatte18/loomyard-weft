```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix set for sibling or twin text left out of step with it (lens fix-consistency).

**Findings**

1. **LOW** — `internal/gatecli/doc.go:908-910` and the `testMapCmd` Long help in `internal/gatecli/testmap.go`.
   - The fixer added a skipped cause, `no Go file tracked at HEAD`, for a named package. It also added a sentence to the `mapPackages` comment.
   - The user-facing twins still list only the two older skip causes: `A package with a modified or untracked Go file, or whose map another call is building, is skipped`.
   - doc.go says `A package with an uncommitted or untracked Go file is skipped naming the file ... so is a package whose build lock ... another call holds`.
   - A reader of the doc or `--help` is not told about the new skip.

2. **LOW** — `internal/testcover/doc.go:10` and the `runner.go` header comment.
   - The `Toolchain` fields `Dir` and `Ctx` are new.
   - `Dir` is the working directory of each go command. `Ctx` kills a command's process group once it is done.
   - doc.go line 10 describes only `BuildTimeout` and `TestTimeout`, and the file header says "each command under its own bound when the toolchain sets one". Neither mentions that a signal context now ends commands.

3. **LOW** — the `mapPackage` doc comment in `internal/gatecli/testmappackage.go`.
   - The comment still reads `The error reports a slot, read or write failure that ends the call.`
   - `mapPackage` now also returns `r.ctx.Err()` after a cancelled build, `lym` build or test. The `mapPackages` comment already covers "a cancelled call".

4. **LOW** — the `runTestMapStandIn` doc comment in `internal/gatecli/testmap_integration_test.go`, and the file header ("stands in for `go`").
   - The comment lists the stand-in's roles: `go test -c` and the coverage binary.
   - The stand-in now has a third role, `go build ... ./cmd/lym` (`runTestMapStandInLymBuild`).
   - The coverage role also appends the `PrebuiltLymEnv` value to a file.
   - The `Hangs` and `Lands` description lines are untouched. These two behaviours are undocumented there.

5. **LOW** — the doc comment of `prebuildLym` in `internal/gatecli/prebuild.go`, and the file header.
   - The header says `prebuild.go builds lym once for a tagged lym gate test run`. `testmap` now calls `prebuildLym` as well.
   - The doc comment says `builds <root>/cmd/lym ... under the run's -p cap`. It does not say that testmap passes a slot environment and a grace.
   - The `env` and `pipeWait` parameters are documented only by the added lines. The header is out of date.

6. **LOW** — the `Hangs` comment in `internal/gatecli/testmap_integration_test.go` (stand-in doc).
   - The comment `A test ending in Hangs never ends` is still accurate.
   - The `gateYAML` comment now says `A test past 20 seconds of 50 milliseconds each times out: 1 second.`
   - That fits, so no change is needed. I raise it only to say I checked it.

No finding of MEDIUM or BLOCKING severity.

**Further tool calls**
1. `grep` of `internal/gatecli/doc.go` and `internal/testcover/doc.go` for skip-cause text, `prebuildLym`, `BuildTimeout`, `cancel` and `Toolchain`. My context did not hold the two doc.go files, and I needed to see whether the module docs cover the new behaviours.
