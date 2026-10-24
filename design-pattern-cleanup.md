# Clean up PATTERN.md: far too many entries

`pattern-cleanup` · Task · enhancement, stencil · running · Darn-Write

## Seen (2026-10-24)

PATTERN.md was already over its headroom line (15651 bytes against 15564, 95% of 16384) before `gate-runs-once` added 510 bytes of entries; that run got through only by shortening 20 other entries, recorded as `changes` decisions.
The overview is inlined in every spawned agent's prompt, so its size is paid on every run.

## Idea (operator, 2026-10-24)

PATTERN has far too many entries and must be cleaned up.

## Review (Sonnet agent, 2026-10-24)

- 97 entries, about 160 bytes each; 79 link a `pattern/` background file (about 131 KB in all); the `[background](...)` links alone cost about 4 KB.
- Proposed: about 30 entries and about 7 KB: 11 kept as they are, 38 merged into 16 survivors, 36 moved to their package's `doc.go`, 5 dropped.
- Bloat: package-local "only package X does Y" rules (about 36, mostly test-enforced with a message naming the owner); one rule split over several entries (shed 4, git 8, panes 5, shuttle 4, config 5, tests 6); rules restated elsewhere.
- Mechanics: `internal/pattern.Check` requires every `pattern/` file to be named by exactly one entry, so a merged or moved entry's background file is merged, moved to `doc.go` or deleted; `(test)` is only a marker and no test reads an entry's text.
- About 189 `.go` files, five contract specs and `burler-cluster-fan.md` cite `PATTERN-` IDs; a removed or merged ID needs a sweep of those messages, and no test checks that a cited ID exists.

Merge groups (survivor: absorbed):
- `cwd-resolution`: told-geometry, hub-containment
- `durable-vs-ephemeral-state`: lyxdirs-single-declarer (keep its `tools/` carve-out)
- `config-layers`: config-strictness, template-overlay, binary-change-reconcile
- `refusal-way-forward`: no-denied-recovery
- `fabric-git`: fabric-destruction-chokepoint, fabric-write-containment
- `push-both-sides`: push-failure-visible (or drop it)
- `pair-teardown`: batten-bookend
- `github-auth`: agent-filed-issues
- `gitrepo-client-boundary`: hub-store-gc, gogit-read-helper, gitexec-checked-call, never-force-add
- `owner-machine`: origin-reconcile
- `shed-recipe-registry`: recipe-setting-lookup, command-row
- `shed-driver`: shed-verb-set, shed-run-directory
- `gate-slot`: gate-self-check-parity, verified-tree, trimpath-safe-root
- `sole-parsers`: ref-shape-registry
- `stencil-ownership`: producer-pointer-rule
- `shuttle-provider-seam`: completion-signal, model-choice-mapping, shuttle-stop
- `shell-guard-table`: shell-mechanics-seam
- `pane-text-injection`: creel-store, orch-pane-single-writer, pause-holds-prompts, guarded-interrupt
- `parent-directive`: role-skills-typed, edit-directive, spawn-color
- `spawn-observability`: log-levels
- `test-speed`: test-tiers, test-economy
- `test-isolation`: testkit, hubforge-fixtures

Kept alone: lyx-temp-root, cli-cobra, cliwire-sole-wiring, fabric-vocabulary, tolerant-records, daemon-wakeups, run-process-owner, leaf-packages.

Move to the owning package's `doc.go`: hub-wide-config, sparse-hub-base, hub-suffix (or keep), mutation-record, parent-lock, shed-producer-seam, transient-stop, recipe-owns-failures, failure-stop-visible, driver-choice-single-site, treadle-runner-seam, plan-generation, glyph-conversion-chokepoint, batcher-registry, review-round, agent-name, friction-capture, pane-binary-resolution, wait-mark-display-only, sandbox-coverage, dev-prod-binary-separation.

Drop: quarry-cgo (in `CLAUDE.md` and enforced by build tests), markdown-link-integrity (`docslink_test` message is clear), comment-line-breaks (enforced by `lyx loom lint-comments`), documentation-lifecycle (in `CLAUDE.md`).

The byte figures are the agent's estimate; the shed and gate groups overlap what `gate-runs-once` changes, so a cleanup runs after it lands.
