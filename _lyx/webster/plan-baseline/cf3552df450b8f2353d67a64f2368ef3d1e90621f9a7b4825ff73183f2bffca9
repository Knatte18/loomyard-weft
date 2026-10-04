# Card 17 — vocab-sweep-docs

**Edit:**
- `README.md`
- `docs/overview.md`
- `docs/sandbox-hub.md`
- `docs/sandbox-howto.md`
- `docs/shared-libs/README.md`
- `docs/shared-libs/lyxcwd.md`
- `docs/shared-libs/configengine.md`
- `contracts/specs/loom-status-spec.md`
- `contracts/recipes/loom-recipe.yaml`
- `crucible/README.md`
- `crucible/review-prompt-template.md`
- `crucible/agents/crucible-reviewer-low.md`
- `crucible/agents/crucible-reviewer-medium.md`
- `crucible/agents/crucible-reviewer-high.md`
- `crucible/agents/crucible-reviewer-xhigh.md`
- `crucible/agents/crucible-reviewer-max.md`
- `plugins/ly/skills/ly-drive/SKILL.md`

**Intent:** Reword the bare `weft`/`warp` occurrences in the listed non-Go files under the scan roots card 18 adds, so the extended scan passes on the swept tree (D10).

Use the Shared Decisions `vocab-sweep` rule, with the fabric module as the noun:
- "the fabric", "the pair", "the code side", "the records side", "fabric's tracked run records";
- or rephrase so no side needs naming.
- `docs/overview.md`'s module table and execution-stack text describe fabric by what it does, pointing at `internal/fabricengine`'s doc for the two-sided design.

`vocab-exempt` identifiers (for example `WeftWorktree` in `docs/shared-libs/lyxcwd.md`'s moved-out list) stay as written.
Measurement reports under `docs/benchmarks/` and `docs/research/` are not reworded; card 18 allowlists them.
D10 also names `contracts/specs/refusal-spec.md` as a hit, but it no longer matches; card 14 owns the stencil claims test.

The ly-drive skill edit ships through `update-plugins.sh` only when the operator runs it.
YAML edits in `loom-recipe.yaml` are comment or description text only; no key, row name or value that code reads changes.
If a hit is a value code reads, report it as a deviation for card 18's exemption list instead.

**ImpactSummary:** Prose rewording across the listed docs, specs, crucible files and one skill; no code path reads the changed text.
