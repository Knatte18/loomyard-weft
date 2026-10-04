# Card 12 — manifest-retire

**Delete:**
- `manifest/roadmap.md`
- `manifest/HANDOFF.md`
- `manifest/designs/curation-triage.md`
- `manifest/designs/fabric-windows-verification.md`
- `manifest/designs/hardener.md`
- `manifest/designs/raddle.md`
- `manifest/designs/reed-mailbox.md`
- `manifest/designs/reed-multi-window.md`
- `manifest/designs/semantic-index.md`
- `manifest/designs/warp-visibility.md`
- `manifest/designs/webster-parallel-execution.md`

**Edit:**
- `CLAUDE.md`
- `README.md`
- `docs/overview.md`
- `docs/code-comment-conventions.md`
- `docs/research/quarry-holistic-fix-log.md`
- `docs/shared-libs/README.md`
- `contracts/specs/loom-plan-spec.md`
- `contracts/stencils/citation_enforcement_test.go`
- `crucible/README.md`
- `crucible/orchestrator-prompt.md`
- `crucible/review-prompt-template.md`
- `internal/boardengine/board.go`
- `internal/boardengine/render.go`
- `internal/fabricengine/doc.go`
- `internal/fabricengine/livestate_doc_test.go`
- `internal/gitrepo/doc.go`
- `internal/shedadapters/doc.go`
- `internal/shedrecipe/coverage_guard_test.go`
- `internal/treadleengine/doc.go`
- `internal/lyxcwd/docslink_test.go`
- `PATTERN.md`
- `pattern/PATTERN-markdown-link-integrity.md`

**Intent:** Delete `manifest/` and fix everything that pointed into it, after card 11 moved its content (D9).

Every link or path citation into `manifest/` is repointed to where its content now lives or dropped:
- design content now in a package doc points at that package's `doc.go` or package comment;
- roadmap items point at the board entry by slug (for example "the board's `doctor` note");
- `loom-plan-spec.md`'s parked parallel-execution link points at `internal/websterengine`'s package documentation;
- `livestate_doc_test.go`'s comment points at `internal/fabricengine`'s doc.

`.claude/agents/mill-done-gate-fixer.md` does not exist in this tree, so there is nothing to fix there.
`tools/mdreflow/reflow_test.go`'s `../manifest/designs/hardener.md` is reflow fixture text, not a link, and stays.
Uses of "manifest" in other senses (fabric's livestate manifest, board lock and manifest patterns) are not this card's.

Lifecycle text:
- `CLAUDE.md`'s "Docs land in the same commit" section and `docs/overview.md`'s Documentation Lifecycle say that an unbuilt design lives in its board entry's body, and a built design lives in its package's `doc.go`.
  The `manifest/` and `manifest/roadmap.md` rules are removed, and CLAUDE.md's task completion rule stops naming the roadmap.
- `docs/overview.md`'s other `manifest/` references are rewritten the same way.

Enforcement:
- `docslink_test.go` scans `docs` only; its comments and the invariant wording drop `manifest/`.
- `citation_enforcement_test.go`'s `citationPrefixes` drops `"manifest/"`.
- `PATTERN-markdown-link-integrity`'s line and background file say `docs/` only.
- The `PATTERN-documentation-lifecycle` line states the new lifecycle rule.

Grep for `manifest/` after the edits.
The only remaining hits are the reflow fixture and the other-sense uses.

**ImpactSummary:** Deletes the manifest folder and repoints its inbound links; the markdown-link scan narrows to docs/ and the citation scan loses one prefix.
