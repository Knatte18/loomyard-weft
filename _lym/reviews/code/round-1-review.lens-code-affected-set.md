```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: I traced everything outside the diff's lines that reads or describes what the diff changes. No member of the affected set is left wrong, so this lens has **no findings**.

**Per change, the set found:**

1. **The guard moves from `cmd/lym/gofmt_test.go` to `internal/testkit/gofmt_test.go`.** Covered.
   - Path references to the old location: the only one was `internal/loomcli/fmtverb.go:105`, and the diff updates it. A search for `TestGofmt` and `gofmt_test` across the repository finds the old path only in `docs/research/*` measurement logs. Those record past runs and are exempt.
   - Guard discovery: `impactset.FindGuardTests` and `ModuleGuardTests` (`internal/impactset/guards.go:83-121`) pick up `//lym:guard` in any package. `gatecli` runs the test through `scanGuards` (`internal/gatecli/scope.go:621-655`) and the diff-scope guard merge (`scope.go:298,314`). The package does not matter to either.
   - Walk root: scankit's `startDir` gives the nearest `go.mod` above `internal/testkit`, which is the repository root (the only other `go.mod` sits under `internal/planglyph/testdata`). So the root and the skip set are unchanged.
   - Package rules: `PATTERN-test-isolation` limits the imports of a kit's non-test files only, and the new file is a `_test.go`. `internal/testkit/testmain_test.go` already runs under `tmuxkit.Main`. `PATTERN-test-only-code` holds, since `gofmtFinding` stays in a `_test.go` file.
   - Package doc: `internal/testkit/doc.go:14` gains a line describing the package's own file. That fits `PATTERN-code-conventions`.

2. **The `//lym:only parent-align finalize` marker is dropped.** Covered.
   - `pattern/PATTERN-test-speed.md`, "Slow whole-repository checks carry `//lym:only`": landing-only applies to whole-repository guards of `cmd/lym`, and "a guard of any package but `cmd/lym`, stays at the task gates". The moved guard now complies, and the section never named the gofmt guard, so nothing there goes stale.
   - The other `//lym:only parent-align finalize` tests are unaffected. The `parent-align` and `finalize` keys have scope `all`, not `marked`, so losing one marked test cannot leave them with nothing to run (`gate.yaml` gates section, `internal/gatecli/only.go:103`).
   - Leaving out `//lym:watch` matches `PATTERN-test-speed`'s rule that a guard over all the module's Go code stays unwatched.

3. **The finding message now names `lym loom fmt <file>`.** Covered.
   - Rules and docs about the verb: the stencils `contracts/stencils/engine/{claude,codex}/edit-directive.md:15` already say "A Go file's formatting is fixed with `lym loom fmt <file>`", and the new message agrees. `docs/overview.md:1296` and `internal/loomcli/doc.go:144` describe the verb without naming the guard's location, so neither is stale.
   - Assertion: the test row in `internal/testkit/gofmt_test.go:76` checks for `"lym loom fmt a.go"`. No other test or fixture asserts the message text.

4. **The `fmt` verb's help text names the new location** (`internal/loomcli/fmtverb.go:105-106`). Covered: a search finds no help golden or test that asserts this `Long` text.

**Rubric items:**
- Declared PATTERN changes: none are declared, so there is nothing to check.
- Stencil wording: the diff changes no stencil.
- Commit messages and departures: the single commit, `19c8c6847`, has no `Pattern-Departure:`, `Loosened-Check:` or `Root-Cause:` trailer. It needs no departure trailer because the move follows `PATTERN-test-speed`. No existing assertion was loosened: the old file's rows all carry over unchanged, and the first row gains one assertion.
- The repository's own rules: no violation found among the PATTERN entries above.

**Tool calls:**
1. Searched for `gofmt` and `go/format` in docs, rules and the tree outside `_lym`: my context lacked any doc or rule reference beyond the earlier search's file list.
2. Searched `_bolt` specs, contracts and internal stencils for `loom fmt` and `ModuleFilesAreFormatted`, and PATTERN files for `internal/testkit`: my context lacked the stencils and the test-only-code rule's text on testkit.
3. Read the `edit-directive.md` stencils' fmt lines and `docs/overview.md:1290-1300`: needed to confirm neither states a guard location or a landing-only check.
