```yaml
lens: code-pattern-fit
outcome: reported
```

Task: the code-pattern-fit lens, checking the diff against the repository's rules.

**Rules checked**
- `PATTERN-gitrepo-client-boundary`: held. The diff only classifies command strings and runs no git.
- `PATTERN-test-speed`: held. `cluster_test.go` keeps its single `//go:build tier0` tag, and the new test is parallel, hermetic and runs no real git. The decision record calls the rows "tier-free", but the file's existing tag is the correct form.
- `PATTERN-refusal-way-forward`: held. The `forkViolation` error message and its way forward are unchanged.
- `PATTERN-shell-guard-table`: not applicable. This is an audit of transcript commands, not a guard table or a pane-shell string.
- `CLAUDE.md` "Docs land in the same commit": held. The fork-discipline paragraph in `internal/burlerengine/doc.go` is updated in the same change. No new module, CLI behavior or cross-cutting invariant appears, so no `PATTERN.md` or `docs/overview.md` edit is needed.
- Declared PATTERN changes: held. The directive's constraints section declares none, and the diff touches no `PATTERN.md` or `pattern/` file.
- Commit messages and departures: held. Neither commit has a `Pattern-Departure:` trailer, and none is needed because the diff departs from no entry. Both commits carry the attribution trailer.
- Comment conventions: held. New doc comments sit above their symbols, begin with the symbol name, and have no end-of-line comments. I did not check line breaks, since `lyx loom lint-comments` alone owns them.
- Markdown semantic line breaks: not applicable, since no `.md` file changed.
- Terminology and filesystem links (`internal/fslink`): not applicable.

No findings.

**Tool calls**
1. `head -3` of `internal/burlerengine/cluster_test.go`, now and at base `04aafa88c`: the diff hunks omit line 1, so my context lacked the tier build tag.
