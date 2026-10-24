```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set, covering what the diff forces outside its touched lines.

**Per change: set found, covered or not**
- Removed or merged PATTERN IDs, citations in live files: a search of all file types outside `docs/research`, `docs/benchmarks`, `testdata`, `_lyx` and `.lyx` found one leftover, F1 below. Everything else still citing `PATTERN-*` names a surviving ID. Covered except F1.
- Deleted `pattern/*.md` background files, path links: every `pattern/PATTERN-*.md` path cited outside `docs/research` resolves. The only misses are `PATTERN-a`, `-b`, `-x` and `-z`, which look like fixtures and were not opened. Covered.
- `docs/overview.md` links: every `PATTERN-*` link in the lines I read points at a surviving file. Covered.
- Test messages and comments redirected to a package doc: "the mutation record section of fabricengine's package doc" resolves to `# The mutation record` at `internal/fabricengine/doc.go:754`. "test economy section" in `cmd/testtiming/redundancy.go:300`, `docs/benchmarks/running-tests.md:97` and `docs/benchmarks/test-redundancy.md:6` resolves to `## Test economy` at `pattern/PATTERN-test-speed.md:118`. Covered.
- Moved rules (`sandbox-coverage`, `dev-prod-binary-separation`, `glyph-conversion-chokepoint`, `agent-name`): their test messages now name the owning package doc by prose, and the sandbox and planparser ones are covered by the diffs I read. The agentname doc was not opened. Covered, unverified for agentname.
- Stencils, `contracts/` and `crucible/`: no removed ID survives in these.

**Findings**
1. LOW, `consistency`, `internal/discussionparser/validate_test.go:258`. The fixture still contains `` `PATTERN-hub-suffix` ``, a removed ID, in a live test file. The acceptance criterion says no live file outside `docs/research/`, `docs/benchmarks/` and `testdata/` cites a removed ID. The string is parser input and not a citation, so a harmless fix is swapping it for a surviving or obviously fake ID.

Tool calls (1):
1. A single Bash call: grep headings in `pattern/PATTERN-test-speed.md` and the fabricengine doc, check every `pattern/PATTERN-*.md` path cited outside the research docs, and grep `docs/overview.md` for `PATTERN-` links. My context lacked these heading and link-resolution checks.
