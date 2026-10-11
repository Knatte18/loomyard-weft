# Card 10 — pattern-overview-trim

**Prosa:**
- `PATTERN.md`
- `pattern/PATTERN-hubforge-fixtures.md`
- `pattern/PATTERN-shed-verb-set.md`
- `pattern/PATTERN-gitrepo-client-boundary.md`
- `pattern/PATTERN-test-speed.md`
- `pattern/PATTERN-test-isolation.md`
- `pattern/PATTERN-glyph-conversion-chokepoint.md`
- `pattern/PATTERN-batcher-registry.md`
- `pattern/PATTERN-pane-binary-resolution.md`
- `pattern/PATTERN-fabric-destruction-chokepoint.md`
- `pattern/PATTERN-leaf-packages.md`
- `pattern/PATTERN-gogit-read-helper.md`
- `pattern/PATTERN-hub-store-gc.md`
- `pattern/PATTERN-ref-shape-registry.md`
- `pattern/PATTERN-shed-run-directory.md`
- `pattern/PATTERN-cliwire-sole-wiring.md`
- `pattern/PATTERN-comment-line-breaks.md`
- `pattern/PATTERN-driver-choice-single-site.md`
- `pattern/PATTERN-failure-stop-visible.md`
- `pattern/PATTERN-testkit.md`
- `pattern/PATTERN-no-denied-recovery.md`
- `pattern/PATTERN-transient-stop.md`
- `pattern/PATTERN-stencil-ownership.md`
- `pattern/PATTERN-review-round.md`
- `pattern/PATTERN-refusal-way-forward.md`
- `pattern/PATTERN-shuttle-stop.md`
- `pattern/PATTERN-shuttle-provider-seam.md`
- `pattern/PATTERN-sole-parsers.md`

**Create:**
- `pattern/PATTERN-push-both-sides.md`
- `pattern/PATTERN-edit-directive.md`
- `pattern/PATTERN-documentation-lifecycle.md`
- `pattern/PATTERN-wait-mark-display-only.md`
- `pattern/PATTERN-spawn-color.md`
- `pattern/PATTERN-hub-wide-config.md`

**Uses:**
- `internal/pattern/check.go`

**Intent:** Shorten the overview to at most 15000 bytes, decision D4, so the next task that adds an entry does not have to cut unrelated wording.
Every entry keeps its name, its trigger, its requirement and its `(test)` mark; no entry is dropped or merged, and no rule changes.
The trimmed entries are exactly those whose line is 230 bytes or longer before this card, as `wc -c` counts a line; every other entry line is left as it is.
Each trimmed line that already carries a link ends at most 200 bytes long, and each of the six that gains one ends at most 230 bytes long, its new link included; at those ceilings the trimmed lines free about 1310 bytes against the 1217 the target needs.
A line shrinks only by losing a clause of mechanism, example or history that its background file now holds: a clause of the requirement itself never leaves the line, so `PATTERN-push-both-sides` keeps all of its obligations.
Detail leaves the entry line for the entry's background file: each trimmed entry with a background file, the files under Prosa beside `PATTERN.md`, gets the detail appended there, and each trimmed entry without one, the six files under Create, gets a new file holding the detail the line loses, and its line gains its `— [background](pattern/<name>.md)` link, whose bytes count toward the target.
`PATTERN-documentation-lifecycle`'s inline link to `docs/overview.md#documentation-lifecycle` moves into its new background file, which keeps pointing there for the lifecycle's full statement.
Some of these background files were already edited by cards 4, 6 and 7; this card appends below their edits and changes none of them.
Multi-byte characters count: the size is bytes, as `pattern.Check` measures it, so the target is checked with `wc -c`.
Every new background file is named by exactly its own entry's link, keeping `Check`'s orphan and link rules green, and the ordering of entries and headings is unchanged, so a reviewer can diff each shortened line against its prior wording for a dropped requirement.

**Verify:** go test ./internal/pattern && [ "$(wc -c < PATTERN.md)" -le 15000 ]
