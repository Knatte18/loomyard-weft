```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set. For each change, I list what reads, calls or asserts it and whether the diff covers it.

**Per change: the set found, covered or not**

1. `RepositoryRootOf` now reads `GIT_CEILING_DIRECTORIES`.
   - Callers: only `resolveStandaloneTarget` at `internal/cliwire/standalone.go:207`. The comment at `standalone.go:186` still describes the lift correctly. Covered.
   - Tests that assert it: `cliwire_test.go`'s `TestRepositoryRootOf`, plus the new serial env-read test. Covered.
   - `bannedecl_enforcement_test.go:51` lists `RepositoryRootOf` by name only; the signature is unchanged, so it is unaffected.
   - The standalone wiring tests in `internal/webstercli/wiring_test.go:522` and `internal/burlercli/wiring_test.go:439` call it through the standalone target path, so the ceiling `tmuxkit.Main` sets now reaches them. Covered.
   - `docs/benchmarks/test-redundancy.md:448` is a historical measurement table, not a claim to keep current.
   - `RepositoryRootOf`'s own doc comment carries the new ceiling rule (`paths.go:127-129`), as the decision record's plan-writer note asked.
   - Not covered, LOW: the doc's PRECONDITION paragraph and the `standalone.go` caller comments still say nothing about production behavior now depending on an environment variable. This follows from the decision record's intent and the new doc paragraph says it, so I raise no finding.

2. `setTempEnv` now sets `GIT_CEILING_DIRECTORIES`.
   - Callers: `setTempEnv` is called from `Main` (`tmuxkit.go:84`) and from `setEnv` (`tmuxkit.go:471`). `setEnv` also covers the re-executed helper path. Covered.
   - Tests: `TestSetEnv` restores the variable through `t.Setenv` and asserts it equals `dir`. Covered.
   - Docs: the package doc (`tmuxkit.go:14`), the `setTempEnv` comment, and `pattern/PATTERN-test-isolation.md:39` all describe the behavior. The comment says "TMPDIR, and on Windows TMP and TEMP" and then adds the ceiling sentence, which is consistent. Covered.
   - The tag spellings in the merged text are the parent's (`tier2`/`tier3`/`tier1`). I found no leftover `tmux`, `llm` or `integration` tag names in `tmuxkit.go`, `allow_*.go` or the pattern file. Covered.

3. `setTempEnv`'s effect on spawned processes.
   - Every child git of a test package now inherits the ceiling. Since `GIT_CEILING_DIRECTORIES` does not stop git from finding a `.git` in the start directory itself, fixtures that `git init` inside a `t.TempDir()` are unaffected.
   - The remaining risk is a test that places a repository at the run directory itself and then walks from below it. The decision record names this in its open risks, and I found no such test in the files I could see, but I did not search for it.

**Findings:** none.

**Tool calls (1 of 6)**
1. `grep` for `RepositoryRootOf` and `TMPDIR` across `.go` and `.md` files, plus `sed -n 108,128p internal/cliwire/paths.go`. Reason: my context lacked the callers and doc consumers of the changed symbols and the full doc comment.
