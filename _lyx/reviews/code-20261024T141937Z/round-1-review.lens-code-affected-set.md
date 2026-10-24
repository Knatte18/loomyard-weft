```yaml
lens: code-affected-set
outcome: reported
```

**Task:** the code-affected-set lens for the amendment round, covering the parent's change to `pattern/PATTERN-test-isolation.md` and what it forces elsewhere.

**Per change, the set found:**
- **The `Main` ceiling clause in `pattern/PATTERN-test-isolation.md` (the parent's one added line):** it is carried into the merged survivor, in the "Private temp directory and leftovers" section. It matches the code that sets it (`internal/testkit/tmuxkit/tmuxkit.go:461`, doc at lines 14 and 458). `internal/testkit/tmuxkit/tmuxkit_test.go:52` asserts it. The `internal/cliwire/paths.go:127-148` walk and its tests at `internal/cliwire/cliwire_test.go:186-274` honor the variable. Covered.
- **The `test-isolation` entry in `PATTERN.md` (line 52) and its `pattern/` link:** the entry text equals the declared text. The set of `pattern/PATTERN-*.md` links in `PATTERN.md` equals `ls pattern/` with no duplicate, so `pattern.Check` should see every file named by exactly one entry. Covered. I did not run `pattern.Check`.
- **Absorbed IDs `PATTERN-testkit` and `PATTERN-hubforge-fixtures`:** the grep over `.go`, `.md` and `.yaml` outside `docs/research/`, `docs/benchmarks/`, `testdata/`, `_lyx/` and `.lyx/` returned no hit. The parent's commit `b9597b2e1` adds no `PATTERN-` ID anywhere except the one background file, so none of its `.go` files cite an absorbed ID. Covered.
- **Lost or contradicted parent rule:** the diff of `pattern/PATTERN-test-isolation.md` between `93f1c8942` and `main` is that single bullet, and it is present verbatim in the merged file. The `Testkit` and `Hub fixtures` sections are intact beside it, with no contradiction. Covered.

**Findings:** none.

**Tool calls (1 of 6):** one grep for the absorbed IDs and for `GIT_CEILING_DIRECTORIES` across the repo, because my context had no repo-wide citation scan or view of the code that sets the variable.
