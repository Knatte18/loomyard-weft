```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering what the `splitShellSegments` change forces outside the lines it touches.

**Per change, the set found:**

- **`splitShellSegments` now drops redirections**
  - The only production caller is `runsMutatingGit` at `cluster.go:108`, which is reached through `firstMutatingGitCommand`.
  - Test callers are `cluster_test.go:397` and `cluster_test.go:454`. The new rows are in the table at `cluster_test.go:380-385`.
  - Covered. The `runsMutatingGit` case on `words[0] == "cd"` reads word 1 of a segment. A redirect now never lands there, so `cd 2>/dev/null dir` resolves correctly.
  - No other package calls the function.
- **Guard description**
  - `doc.go:301-305` carries the new clause, and the `splitShellSegments` doc comment carries its own line at `cluster.go:358`. Both are covered.
  - The `firstMutatingGitCommand` doc comment at `cluster.go:81` still reads "split into segments at ;, &, | and newlines". That is still true, because it names segments rather than words, so nothing is left wrong.
  - The CLAUDE.md docs rule asks for `docs/overview.md` and `PATTERN.md` only when a module is added, CLI behaviour changes or a new invariant appears. None applies here, so covered.
- **Stencils**
  - `contracts/stencils/burler/burler-template-reviewer.md:24` only says the reviewer runs no mutating git command. It does not describe the guard's parsing, so it is unaffected.
  - The other stencil hits matched `2>/dev/null` or "mutating git" in unrelated text. The `docs/research/` hits are historical records and are not rule documents.
- **PATTERN and `pattern/`**
  - No entry covers the lexical guard. `PATTERN-shell-guard-table` governs `internal/shellguard`, which this code does not touch.
  - The diff declares no PATTERN changes, and the directive's declared-changes list is empty. Nothing to land.
- **Commit message**
  - It carries no `Pattern-Departure:` trailer and needs none, because no PATTERN entry is in play.
  - Its claims match the diff.
- **Other consumers of the guard's output**
  - `forkViolation` still returns an error for a mutating command, and I made no change to it, so D3 holds.

**Findings:** none.

**Tool calls (2 of 6):**
1. `grep` for other callers of `splitShellSegments` and `firstMutatingGitCommand`, and for docs and stencils that mention the guard. My context lacked a repo-wide caller search.
2. `grep` of `burler-template-reviewer.md` for guard wording. My context lacked the stencil text.
