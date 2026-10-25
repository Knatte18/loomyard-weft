```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, listing per change what the diff forces outside its own lines.

**Affected set per change**

- **New `report.md` under `docs/research/creel-hook-delivery-2026-10-25/`.**
  - It is a new file, so it changes no symbol, key or output.
  - Nothing in the repo reads it or enumerates the `docs/research/` entries.
  - `internal/lyxcwd/enforcement_test.go:565` only lists `docs/research/` as a protected path, with the reason that measurement reports record one run and are not rewritten. That fits this report and needs no edit.
  - `internal/lenses/template.yaml` and `config_test.go` cite `docs/research/session-fork-spike.md` only.
  - `crucible/review-prompt-template.md:72` names `docs/research/&lt;module&gt;-*.md` as a pattern, not a list.
  - Covered, with nothing left wrong.
- **The two new `.tsv` logs.**
  - Sibling research directories already hold committed `.tsv` files (`batch-budget-trial-2026-10-24/forks.tsv` and `runs.tsv`), so the file type fits convention.
  - No Go code or fixture reads either log.
  - Covered.
- **Docs landing rule from `CLAUDE.md`.**
  - The diff adds no module, no CLI behavior change and no cross-cutting infrastructure.
  - So `doc.go`, `docs/overview.md` and `PATTERN.md` have nothing to land.
  - Covered.
- **Stencils.** The diff changes none, so the "Stencil wording" rule does not apply.
- **Report references to existing repo artifacts.**
  - It cites the note `creel-carry-agent-mail`, the note `creel-agents`, and the verbs `lyx creel post --reply`, `lyx creel interrupt` and `inject --interrupt`.
  - I did not check that those verbs exist in the current CLI. The report's own text says the stub logged the reply and that the real `lyx` refused outside a hub, which suggests `post --reply` exists.
  - The report also cites the Codex spike at `docs/research/codex-spike-2026-10-24/report.md`, and that file exists.
  - No finding.

**Findings:** none for this lens.

**Tool calls (2 of 6)**
1. Grepped the repo for references to `docs/research` and listed `docs/research/`, to find readers or enumerators of the directory that the manifest lacked.
2. Grepped the matching lines in the four hits, and listed `codex-spike-2026-10-24` and `batch-budget-trial-2026-10-24`, to confirm the path rule and the `.tsv` precedent.
