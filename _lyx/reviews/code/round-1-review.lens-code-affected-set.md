```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set. For each change, I looked for every caller, reader and assert of the removed or merged `PATTERN-` IDs and `pattern/` files, and checked whether the diff left any of them wrong.

**List: change → set found → covered?**
- **Removed or merged `PATTERN-` IDs in Go, `contracts/`, `crucible/` and `docs/` (outside `docs/research/` and `docs/benchmarks/`):** I swept every `PATTERN-[a-z0-9-]+` token repo-wide, excluding `docs/research/`, `docs/benchmarks/`, `testdata/`, `.lyx` and `_lyx`. Each one is in `PATTERN.md`, except test placeholders (`PATTERN-a`, `PATTERN-x`, `PATTERN-old-rule`, `PATTERN-late-gone` and similar). Covered.
- **Test messages and `//testtiming:keep` reasons changed in the 18 overlap files (`cmd/lyx/*_test.go`, `contracts/stencils/*_test.go`, `internal/planparser/shape_enforcement_test.go`, `cmd/testtiming/redundancy.go`):** each cites a surviving ID or names the owning package doc. No stale cite left. Covered.
- **Deleted `pattern/PATTERN-*.md` files (absorbed, moved and dropped):** every `pattern/PATTERN-*.md` path referenced in live files, outside `docs/research/` and `docs/benchmarks/`, resolves. The only misses are the placeholders `PATTERN-a`, `PATTERN-b`, `PATTERN-z` (`internal/pattern/check_test.go`) and `PATTERN-x` (`internal/landingshed/mergerecord_test.go`), which are fixtures. Covered.
- **Parent's tier-tag rewrites (`integration`, `tmux`, `llm` → `tierN`) in the removed `test-economy`, `testkit` and `hubforge-fixtures` files:** they appear in the survivors `pattern/PATTERN-test-speed.md` and `pattern/PATTERN-test-isolation.md` (isolation lines 14, 27–28, 47–48, 84; speed lines 19, 40–43, 49, 54–72, 83, 103, 141). I found no old spellings left. Covered.
- **Parent's new `PATTERN-test-speed.md` text about `cmd/lyx/tiertags_test.go` and the `//lyx:guard` rule:** present at line 19. Covered.
- **`cmd/testtiming/redundancy.go` report string:** it now points at `pattern/PATTERN-test-speed.md`'s test economy section, and that survivor has the section (the line 141 coverage text is there). Covered.
- **Docs and counts (`docs/overview.md`, `CLAUDE.md`):** I found no stale entry counts, no `about a hundred`, and no references to removed entries in `docs/`, `CLAUDE.md`, `PATTERN.md` or `internal/pattern`. Covered.
- **Moved rules:** I did not compare each `doc.go` section against its original background file. That is the handler's holistic review, not this lens.

**Findings:** none.

**Tool calls (2, of 6 allowed):**
1. Bash: resolved every cited `pattern/PATTERN-*.md` path, outside `docs/research/` and `docs/benchmarks/`. The manifest only covered `PATTERN-` IDs, not file paths.
2. Bash: grepped for stale entry-count phrasing and for removed ID names inside `PATTERN.md` and `pattern/`. The manifest did not show the survivors' text for these.
