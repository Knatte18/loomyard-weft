```yaml
lens: code-pattern-fit
outcome: reported
```

Task: the code-pattern-fit lens, checking the diff against PATTERN.md, the `pattern/` background files and CLAUDE.md.

**Rules checked**
- `PATTERN-fabric-git`, seeding stays in `internal/fabricengine`: **held**. `seedWeftArtifactExcludes` goes through `mutateGitExclude` into `.git/info/exclude`. No `.gitignore` is written, no raw git runs, and nothing sits outside the package.
- `PATTERN-durable-vs-ephemeral-state`, `.lym/` never tracked, name from `lymdirs`: **held**. The seeded pattern is `lymdirs.DotLymDirName + "/"` and the new test builds its path from `lymdirs.DotLymDirName`. The literal `"logs"` in `bolt_integration_test.go` names a subdirectory of a scratch fixture and is not a production path, so I raise no finding.
- `PATTERN-gitrepo-client-boundary`: **held**. The diff adds no git invocation. The test only calls `gitkit.LsFiles`.
- `PATTERN-test-isolation`, hub fixture stays `hubforge`: **held**. The scenario keeps `hubforge.CopyHub`, and the diff adds no `hubforge` import to a package in `fabriccli`'s dependency set.
- `PATTERN-test-speed`, `t.Parallel` by default and process-global state named: **held with one note**, see the finding below.
- Declared PATTERN changes: **held**, nothing to land. The decision record declares no PATTERN entry change, and the diff touches neither `PATTERN.md` nor `pattern/`.
- Commit messages and departures: **held**. None of the three commits carries a `Pattern-Departure:` trailer, and none is needed.
- CLAUDE.md "Docs land in the same commit": **held**. The `doc.go` sentence and the `bolt.go` and `recordsgit.go` doc comments land with the behavior change. A new invariant would need a PATTERN entry, and this change adds none.
- Comment conventions (`golang-comments`):
  - **held**: the new `Commit` and `commitWeftAt` doc comments are self-contained and name no other symbol.
  - **held**: the `clone.go` rewrite carries no end-of-line comment, no history and no comparison.
  - **held**: the new test comment is above-line.
- CLAUDE.md "Production only via `update-plugins.sh`", "Worktrees", "Notes go in git": **held**, not touched.

**Findings**

1. LOW, `consistency`, `internal/fabriccli/hubmutation_integration_test.go:57-72`.
   - The step now asserts on `logger.TraceDir()` and resets the sink in `t.Cleanup`, both process-wide state.
   - The scenario runs under `t.Parallel`, as the file's doc comment says.
   - `PATTERN-test-speed` says a test touching process-global state names that. The comment on the `t.Cleanup` line explains why the step runs from the board worktree, but not the global reset.
   - The unconditional reset to `""` could also clobber an override another parallel test in the binary has set, and the assertion could read one.
   - I did not check whether any other test in `fabriccli_test` sets the sink, so this is a risk to check, not a confirmed defect.

**Tool calls**
- None made. Everything I needed was already in the inherited context.
