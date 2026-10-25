```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: what the diff's changes force outside the lines it touches.

**Per change, the set found:**
- Signatures, keys, messages and behaviours: the diff adds 33 new files under `docs/research/codex-review-bench-2026-10-25/` and touches no existing file. It changes no Go code, signature, stencil, config or PATTERN entry. Callers, tests and stencils affected: none, covered.
- New report directory: nothing indexes the reports. `docs/overview.md` lists no research reports, and the other `docs/research/*` entries are standalone. The only references to `docs/research` are comment cites of specific older reports, plus `internal/lyxcwd/enforcement_test.go:565`, which lists `docs/research/` as a path of measurement reports that are not rewritten. Nothing needs a link or update, covered.
- Docs-in-same-commit rule (`CLAUDE.md`): the diff adds no module, CLI behaviour or cross-cutting infrastructure, so there is no `doc.go`, `docs/overview.md` or `PATTERN.md` update to owe. Covered.
- Stencil wording: the diff changes no stencil. `data/harness/build.sh` reads stencils from each snapshot's own `contracts/`, so no live stencil is affected. Covered.
- Fixtures and scanners: no test scans `docs/research/` content, so the new files break none. Covered.

**Findings:** none from this lens.

**Tool calls (2):**
1. A grep for `docs/research` across the repo's md, go and yaml files, plus an `ls` of `docs/research`. It checked for an index or scanner that must list the new directory, which my context lacked.
2. A grep for `docs/research` in `internal/lyxcwd/enforcement_test.go`, `internal/lenses/config_test.go` and `internal/lenses/template.yaml`. It checked whether any test asserts on that path, which my context lacked.
