```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. I mapped everything outside the touched lines that the diff's changes affect.

**Per change, the affected set and whether it is covered**

1. **`runCheckingGitEntries` takes a new `snapshot` parameter.** Callers are the two `Main` call sites (`internal/testkit/tmuxkit/tmuxkit.go:133`, `:144`), both updated, and `TestRunCheckingGitEntries`, also updated. A repo-wide grep finds no other caller. Covered.
2. **`checkNewGitEntries` takes a new `sightings` parameter and new exit-code semantics.** Its only callers are `runCheckingGitEntries` and `TestCheckNewGitEntries`, both updated. Covered.
3. **The finding text changes and "appeared during the run" is dropped.** A grep for the old wording and for `/tmp/.git` across `*.go`, `*.md`, `*.tmpl` and `*.yaml` (outside `_lym`, `.lym` and `.scratch`) finds no stencil, spec, fixture or doc that reads or quotes it. Only the tmuxkit tests assert it, and they are updated. Covered.
4. **The `.git` check's blame rule changes.** Three rule documents describe it:
   - the tmuxkit package doc (`tmuxkit.go:37-46`): updated;
   - the background `pattern/PATTERN-test-isolation.md:105-119`: updated;
   - the `PATTERN-test-isolation` one-line entry ("hermetic git and `tmuxkit.Main`…"): it names no blame rule, so it stays accurate.

   Covered. The directive declares no PATTERN changes, so "Declared PATTERN changes landed" raises nothing.
5. **tmuxkit gains a production import of `internal/fswatch`, which brings `internal/logger` in through it.**
   - The `internal/fswatch` section of `PATTERN-leaf-packages` limits only what fswatch imports, and fswatch is unchanged.
   - tmuxkit has no leaf section of its own: the grep of `pattern/PATTERN-leaf-packages.md` matched only `internal/fswatch`.
   - The testkit import rules in `internal/testkit/enforcement_test.go` deny `*cli`, `os/exec`, `gitexec` and `gitkit`, and fswatch and logger are none of these.
   - `logger`'s `init` only reads the `LYM_LOG_*` variables (`internal/logger/logger.go:300-318`), so the import has no new side effect on test binaries.

   Covered.
6. **`internal/fswatch/testmain_test.go` moves to `package fswatch_test`.** The Tmux Test Isolation enforcer groups test files by directory, not by package name (`cmd/lym/tmuxisolation_test.go:205-223`), so an external-package `TestMain` that calls `tmuxkit.Main` still satisfies it. The other test files stay `package fswatch` (`leaf_enforcement_test.go:4`, `fswatch_test.go:3`), and Go allows an internal and an external test package side by side. The new header comment matches the import cycle it explains. Covered.
7. **Commit messages and departures.** Both `Loosened-Check:` trailers on 4d3bbbc2 name the test and the loosened behaviour, and the cause they give is measured (22 gate runs between 2026-11-06 and 2026-11-18). No `Pattern-Departure:` trailer is present. Covered as far as this lens goes; whether the measurement is true belongs to the handler's claims check.
8. **Stencil wording.** No stencil or spec is changed, and none reads the changed output. Nothing applies.

**Findings:** none.

**Tool calls:**
- I grepped the repository for the old message wording, `/tmp/.git` and the changed function names, and grepped the TestMain scanner's package grouping in `cmd/lym/tmuxisolation_test.go`. My context held no repo-wide search for readers of the changed output, and not the enforcer's grouping rule.
- I grepped `internal/logger`'s package-level vars and `init`, and the testkit enforcement test, for logger and fswatch. My context lacked logger's import-time side effects and the testkit deny list's coverage of the new import.
