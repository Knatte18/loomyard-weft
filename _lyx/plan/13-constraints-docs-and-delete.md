# Card 13 — constraints-docs-and-delete

**Edit:**
- `CLAUDE.md`
- `README.md`
- `docs/overview.md`
- `docs/benchmarks/fixture-copy.md`
- `docs/benchmarks/running-tests.md`
- `docs/research/linux-portability-survey.md`
- `docs/research/session-fork-spike.md`
- `docs/shared-libs/configengine.md`
- `docs/shared-libs/lyxcwd.md`
- `contracts/specs/llm-model-spec.md`
- `contracts/specs/shed-recipe-spec.md`
- `crucible/README.md`
- `crucible/orchestrator-prompt.md`
- `crucible/review-prompt-template.md`
- `plugins/ly/skills/ly-drive/SKILL.md`
- `internal/lyxcwd/docslink_test.go`

**Delete:**
- `CONSTRAINTS.md`

**Intent:** Repoint the remaining `CONSTRAINTS.md` references in docs, specs, crucible and the ly-drive skill, then delete `CONSTRAINTS.md` (D7).

`CLAUDE.md`'s first section becomes "`PATTERN.md` is authoritative".
It says to read `PATTERN.md` and the `pattern/` background files that touch a change before writing or reviewing code, that lyx-spawned agents get the overview inlined, and that a new cross-cutting invariant goes into PATTERN in the same commit as the code.
`CLAUDE.md`'s "Docs land in the same commit" list replaces "`CONSTRAINTS.md` for a new invariant" with PATTERN.

Every other reference names the PATTERN entry (`PATTERN-<name>` from the `entry-map` table) or `PATTERN.md`, as a markdown link where the source file uses links.
`docs/overview.md`'s `../CONSTRAINTS.md#package-naming` link points at `../pattern/PATTERN-cli-cobra.md`, which holds the package-naming rule.
`docslink_test.go`'s allowlist entry for it is removed, because the link now resolves.
A stale allowlist entry fails the scan, so the removal is required.

Measurement reports under `docs/benchmarks/` and `docs/research/` get the same mechanical repoint.
A bare filename mention changes to `PATTERN.md`; the recorded results are left untouched.
The ly-drive skill edit ships through `update-plugins.sh` only when the operator runs it, and plugins stay at `1.0.0`.

After the edits, `grep -rn CONSTRAINTS` over the tracked tree, excluding `_lyx/` and `.lyx/`, finds nothing.
Any straggler joins this card.

**ImpactSummary:** Repoints the remaining doc/spec/skill references and deletes CONSTRAINTS.md; one docslink allowlist entry retires with the broken link it covered.
