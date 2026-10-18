# Audit: ideas from tree-sitter-analyzer for quarry and its use in lyx

`tsa-quarry-audit` · Note · audit, quarry, webster, burler, loom

## What was studied

On 2026-10-18 an agent read tree-sitter-analyzer (TSA), a Python project the operator downloaded to `/home/knatte/Code/tree-sitter-analyzer`, against quarry's docs and lyx's use of quarry, to find ideas worth porting to Go.
It read TSA's code and docs only, ran nothing.
Below is the agent's finding, then the orch's view.

Framing: quarry already has `start`/`sigend`/`end`, `signature`, `doc`, a `delta` marking `body`/`signature`/`doc`/`file` changes, and since v0.3.0 `enclose` (path:line to its enclosing glyph); lyx still pins v0.2.1.
Quarry's own measurements found reference and definition tools no better than grep, and a lossy compact view cut precision from 0.96 to 0.82 (`quarry/wts/quarry/docs/rewrite-plan.md` §2).
TSA's surface (8 MCP facades, 361 CLI flags, 22 language plugins) is large, but much of it already exists in quarry or lyx.

## Candidate ideas

### 1. Inline symbol bodies in the card pack (medium-small)

- TSA: `tree_sitter_analyzer/mcp/tools/symbol_body_inline.py` with `call_path_enrich.py` inlines the verbatim body beside each symbol location, on the argument that cost is turns times tokens, so a follow-up Read per file:line costs more than a bigger reply; bodies are capped in three tiers, with a `full_at` file:line pointer when cut.
- Today: `internal/planglyph/pack.go` (`KickstartPack`) resolves glyphs, but `lyx loom card-pack <N>` lists spans only, so the fork spends a parallel read turn per card (`internal/websterengine/doc.go`, about line 170).
- Gain: the Webster implementer saves one turn per card; content stays fresh since card-pack runs at the card's start; a size cap with a pointer keeps large types bounded.
- Caveat: quarry §2 says turn savings did not reliably show, so measure with the ladder first.

### 2. Bump quarry to v0.3.0 and wire `enclose` (small)

- TSA: `codegraph_ast_path` ("what is at file:line?") and the `pr` route's diff-to-symbol mapping (`mcp/tools/codegraph_pr_review_tool.py`).
- Today: quarry v0.3.0 ships `Enclose` in its facade and CLI; lyx is on v0.2.1 with no wrapper.
- Gain:
  - Burler and reviewer findings, test failures and `go vet` lines could be mapped to glyphs mechanically, so findings and fix cards cite glyphs instead of line numbers that go stale.
  - `ScopeGuard` (`internal/planglyph/scope.go`) could map edits outside any symbol, such as file headers and imports.
  - A `lyx quarry enclose` verb.

### 3. Give reviewers the symbol delta (small-medium)

- TSA: `codegraph_pr_review_tool.py` and `ast_diff.py` classify each change as signature, body, added, removed or renamed, plus a blast radius.
- Today: quarry's `delta` already reports this; in lyx only `recordbatch.go` uses it (done checks, handle binding, `ScopeGuard`, `DetectDrift`), and reviewers read the textual diff. Quarry §7 lists "a review gate reads a symbol-table change" as a planned use.
- Gain:
  - Render the batch delta (created, deleted, modified with which part, rename tiers) into the Burler and review prompts.
  - Flag exported members whose signature changed but which the plan does not name as a target, as a reviewer focus list.

### 4. A typed reference query in quarry, `impact` or `assert-no-callers` (medium-large)

- TSA: `nav callers/callees/impact` (`callers_tool.py`, `codegraph_impact_tool.py`). The useful part is the honest-truncation contract in `docs/agent-envelope-contract.md`: a `truncated` flag, the exact total before the cap, the count listed with its cap, and a `next_step` stating the real numbers. Its name-based resolution is the silent pick quarry forbids.
- Today: quarry parks this as phase 2; lyx runs its own Go-only typed reference finder (`internal/planglyph/callercoverage.go`, `typesload.go`) for the `caller-uncovered` and `delete-before-reference` checks.
- Gain:
  - The plan writer could ask for the callers of a Delete or re-sign target before writing cards, instead of hitting `caller-uncovered` at the plan gate and taking a rework loop.
  - Webster and reviewers get the same answer.
  - It becomes the hook for Python and C# later.
  - Keep `verified: true` and the four-part truncation shape.

### 5. A doc-drift scan that resolves references in docs (small-medium)

- TSA: `doc_sync.py` (`--doc-sync`) finds backtick spans and link targets in markdown that no longer exist.
- Today: nothing in lyx resolves glyph or path references in `docs/*.md`, `PATTERN.md` or `doc.go`; quarry §7 lists "documentation drift" and "every exported symbol a doc" as mechanical uses.
- Gain: a `go test` scan, or a landing step, that runs every glyph-shaped or path-shaped backtick token in docs through one batched `Resolve`, enforcing "docs land in the same commit" mechanically after renames and deletions.

### Later: TSA's per-language extractors as reference for the Python and C# alphabets (large)

- TSA: `tree_sitter_analyzer/languages/`, `queries/`, `synapse_resolver/`.
- Use: only as a cross-check for the gaps quarry §6 lists (Python nested classes, C# `partial`, overloads, fields and properties); their naming is not glyph-unique, so the code is not to be ported.

## Attractive but would not pay off for lyx

- Affected tests (`test_map`, `--affected`): `internal/impactset` already derives the reverse-dependency test set exactly through `go list`.
- Signatures-only and other compact views (`structure action=signatures`, `summary_only`, `count_only`): quarry measured that lossy views cost precision, and the `glyphs` view is already the cheap index.
- TOON output format: about half the bytes, but lyx agents copy glyphs verbatim, and JSON or quarry's text view is already compact.
- BM25 or semantic symbol search: the `glyphs` index plus grep covers it, and quarry measured LSP-shaped symbol tools no better than grep.
- Verdict envelope and `agent_summary.next_step`: lyx gates are mechanical, with check IDs and blocking or informational severity; a second verdict alphabet only duplicates them.
- Architectural constraint DSL (`architectural-constraints.yml`): PATTERN invariants are already enforced by `go test` scans.
- SQLite/FTS5 index, SHA-256 incremental sync, caching: quarry answers the whole repository in 0.24 to 0.6 s with no cache, and a cache adds staleness risk to gates.
- Health grades, complexity heatmap, dead code, clones, UML/Mermaid: none feeds a plan, gate or card decision.
- `mutation_probe` ("does this test constrain this code?"): interesting for reviewers, but large, and it runs tests per mutation.
- `co_change` git coupling: a probabilistic hint for planners, and quarry deliberately avoids scores and thresholds.
- Decision journal: lyx already has the discussion record and the board.
- Partial read by line range: the Read tool already does it.
- Curated skills that limit the tools an agent sees: lyx already scopes each agent's tools in its prompt.

## The agent's top three, in order

1. Bump to quarry v0.3.0 and wire `enclose` into lyx (`lyx quarry enclose`, glyph-cited reviewer findings, `ScopeGuard`): small, and the quarry side is done.
2. Give reviewers the batch's symbol delta, plus a flag on undeclared signature changes: the data is already computed in `recordbatch.go`.
3. Inline bodies in `lyx loom card-pack` with size caps and a `full_at` pointer, measured with the ladder before it becomes the default.

It rates the typed `impact` query (idea 4) the most valuable longer-term item, mainly once a second language exists.

## Orch view, not a decision

- Idea 1 is a variant of the kick-start pack, which the glyph audits in `docs/research/glyph-audit-2026-10-12` to `-17` found saved no exploration, and which the operator decided to remove (note `kickstart-pack-remove`); it should not be tried again without a new reason.
- Idea 2 overlaps note `enclose-checks`; its glyph-cited findings are a use that note does not name yet.
- `internal/impactset` is relevant to note `verify-diff-scope`.
- For Go references, gopls answers exactly from type information, where tree-sitter matches names.
