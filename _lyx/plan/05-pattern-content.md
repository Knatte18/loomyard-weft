# Card 5 — pattern-content

**Create:**
- `PATTERN.md`
- `pattern/PATTERN-cwd-resolution.md`
- `pattern/PATTERN-told-geometry.md`
- `pattern/PATTERN-durable-vs-ephemeral-state.md`
- `pattern/PATTERN-hub-containment.md`
- `pattern/PATTERN-hub-suffix.md`
- `pattern/PATTERN-cli-cobra.md`
- `pattern/PATTERN-cliwire-sole-wiring.md`
- `pattern/PATTERN-config-strictness.md`
- `pattern/PATTERN-refusal-way-forward.md`
- `pattern/PATTERN-fabric-vocabulary.md`
- `pattern/PATTERN-fabric-git.md`
- `pattern/PATTERN-fabric-destruction-chokepoint.md`
- `pattern/PATTERN-mutation-record.md`
- `pattern/PATTERN-pair-teardown.md`
- `pattern/PATTERN-batten-bookend.md`
- `pattern/PATTERN-gitrepo-client-boundary.md`
- `pattern/PATTERN-shed-verb-set.md`
- `pattern/PATTERN-shed-run-directory.md`
- `pattern/PATTERN-transient-stop.md`
- `pattern/PATTERN-driver-choice-single-site.md`
- `pattern/PATTERN-plan-generation.md`
- `pattern/PATTERN-ref-shape-registry.md`
- `pattern/PATTERN-glyph-conversion-chokepoint.md`
- `pattern/PATTERN-gate-self-check-parity.md`
- `pattern/PATTERN-verified-tree.md`
- `pattern/PATTERN-review-round.md`
- `pattern/PATTERN-sole-parsers.md`
- `pattern/PATTERN-agent-name.md`
- `pattern/PATTERN-stencil-ownership.md`
- `pattern/PATTERN-friction-capture.md`
- `pattern/PATTERN-completion-signal.md`
- `pattern/PATTERN-shuttle-provider-seam.md`
- `pattern/PATTERN-pane-binary-resolution.md`
- `pattern/PATTERN-spawn-observability.md`
- `pattern/PATTERN-leaf-packages.md`
- `pattern/PATTERN-testkit.md`
- `pattern/PATTERN-hubforge-fixtures.md`
- `pattern/PATTERN-test-tier-purity.md`
- `pattern/PATTERN-hermetic-git-tests.md`
- `pattern/PATTERN-tmux-test-isolation.md`
- `pattern/PATTERN-quarry-cgo.md`
- `pattern/PATTERN-markdown-link-integrity.md`
- `pattern/PATTERN-comment-line-breaks.md`

**Uses:**
- `CONSTRAINTS.md`

**Intent:** Write loomyard's PATTERN from `CONSTRAINTS.md`, under the Shared Decisions `entry-map` and `pattern-text` rules and D1's format.

`PATTERN.md` opens with `# PATTERN` and one sentence of framing.
The `##` topic headings follow the `entry-map` table's grouping, and every entry is exactly one bullet line:
`` - `PATTERN-<name>` — <when it applies and what it requires>[ (test)][ — [background](pattern/PATTERN-<name>.md)] ``.
- Each line names its trigger and its requirement, so a reader can judge relevance from the line alone.
- `(test)` marks an entry that a `go test` enforcement test checks, by the facts in its `CONSTRAINTS.md` section.
- Aim for about 150 characters per line, never over 200 (hard cap 320), so the overview lands near 9–11 KiB, well under the 16 KiB cap card 6 enforces.

Each listed background file carries what does not fit the line: the section's remaining bullets, owner sets, carve-outs, rationale, edge cases and the names of the enforcing tests.
A background file's first line is `# PATTERN-<name>`.
Exactly the entries whose rows in the `entry-map` table say "background" link to one; every other entry is complete as written.
For a merged entry (`leaf-packages`, `sole-parsers`), the background file keeps each merged package's own rule under its own subheading, so no clause of a merged section is lost.

The content states the invariants as they hold at this commit.
Later cards update the entries their own changes touch:
- card 12: `markdown-link-integrity` and `documentation-lifecycle`;
- card 18: `fabric-vocabulary`;
- card 20: `agent-name`.

The `pattern-text` rule binds every line.
Rewording `CONSTRAINTS.md` text that addresses agents or the dev tooling (the `scribe:*` skill references in the comment line-break section, the ly-drive driver in the transient-stop rationale) is part of this card, not a later one.
`CONSTRAINTS.md` stays in place until card 13.
