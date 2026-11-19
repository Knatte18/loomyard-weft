# Card 20 — crucible-docs

**Delete:**
- `crucible/README.md`
- `crucible/orchestrator-prompt.md`
- `crucible/review-prompt-template.md`
- `crucible/agents/crucible-reviewer-low.md`
- `crucible/agents/crucible-reviewer-medium.md`
- `crucible/agents/crucible-reviewer-high.md`
- `crucible/agents/crucible-reviewer-xhigh.md`
- `crucible/agents/crucible-reviewer-max.md`

**Edit:**
- `internal/fabricvocab#docRoots`
- `internal/hardenshed/doc.go`
- `tools/codestats/main.go`
- `tools/codestats#matchesAny`

**Prosa:**
- `docs/overview.md`
- `docs/reference/weaving-glossary.md`
- `docs/skills.md`
- `pattern/PATTERN-fabric-boundary.md`

**Uses:**
- `internal/fabricvocab/doc.go`

**Intent:**
Delete `crucible/` and move its durable method into lym (D12).

Before deleting, read the crucible files and carry their method into `internal/hardenshed`'s package doc, in its own section:
- Why behaviour review differs from text review.
- Why rounds are independent and verified by Go, never by a round's own "merge-ready".
- The merge bar: one green live run with zero stray state, the concurrent run a diagnostic amplifier and never a merge gate.
- What a converged campaign does and does not prove.
The operational rules already live in the `harden-rubric-review` stencil (card 12).

Then delete every file under `crucible/`.
`fabricvocab.docRoots` drops `crucible`, since the directory no longer exists.

`docs/overview.md` drops its Tenter and crucible wording, the tree line and bullet naming `treadleengine`, and the links to crucible's files, and points at `internal/hardenshed` and the `hardener` recipe.
`docs/reference/weaving-glossary.md` drops the `tenter`, tenterhooking and tenter-as-agent-role rows and the "`hardener` and `tenter`" note; the treadle row it holds as a loom term becomes free, and `hardener` points at the recipe and the package.
`docs/skills.md` drops any Hardener-draft wording, and `PATTERN-fabric-boundary.md` drops `crucible/` from its doc roots.
`tools/codestats`' file comment usage example and `matchesAny`'s doc comment name `crucible` as an `-exclude` value; both take another directory name, such as `docs/research`.
After this card no doc names crucible or Tenter.
Three kinds of mention stay, since none is a live path or a doc of lym's current method:
- comments that cite a crucible round as history (`internal/planparser`, `internal/planglyph`, `internal/shuttleengine/attach.go`), which record where a rule came from;
- the `TestMergeCrucible_*` test names in `internal/fabricengine`, which name a merge scenario;
- the measurement and research records under `docs/benchmarks/` and `docs/research/`, which record past runs at a point in time.

**ImpactSummary:** Deletes a doc tree and one doc-root name; the docs and the hardenshed package doc are reworded, with no code behaviour change.
