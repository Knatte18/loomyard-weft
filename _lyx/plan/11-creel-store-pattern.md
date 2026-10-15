# Card 11 — creel-store-pattern

**Create:**
- `pattern/PATTERN-creel-store.md`
- `internal/creelengine/seam_enforcement_test.go`
- `plan:internal/creelengine#TestCreelStore_DeliverySeamImports` -> `func TestCreelStore_DeliverySeamImports(t *testing.T)`

**Edit:**
- `//PATTERN.md`
- `//pattern/PATTERN-told-geometry.md`
- `//cmd/lyx/testdata/test-budget.yaml`

**Intent:** Land the new cross-cutting invariant `PATTERN-creel-store` (D20) with its import scan.
`PATTERN.md` gains, under "Agents and prompts", this entry line verbatim:

```markdown
- `PATTERN-creel-store` — Writing the creel journal or reaching a pane from creel: only the creel daemon in `internal/creelengine` writes it, and that engine reaches a pane only through its `Delivery` seam. (test) — [background](pattern/PATTERN-creel-store.md)
```

`PATTERN.md` stands 9 bytes under its headroom line (`pattern.HeadroomBytes`), so the same edit shortens the wording of other entry lines, never dropping a rule, moving any cut detail into that entry's background file where it has one, until the file is no larger than it was before this card; the per-card PATTERN headroom lint fails it otherwise.
The cuts come from these entries, each with a background file to take the cut detail: `PATTERN-edit-directive` (the orch-stencil exemption), `PATTERN-glyph-conversion-chokepoint` (`planparser`'s external-test unit mapping), `PATTERN-sole-parsers` (the parser list, kept in full in its background), `PATTERN-hub-containment`, `PATTERN-durable-vs-ephemeral-state` and `PATTERN-parent-directive`, taken in that order until the size holds.
No test pins the wording of an entry line in the repository's `PATTERN.md`; the tests that read a `PATTERN.md` build their own fixture.
`pattern/PATTERN-creel-store.md` states the rule and its two halves: the journal is written only by the creel daemon, which holds because the journal writer is unexported in `internal/creelengine` and the daemon's `Run` is its only caller, checked by review; and `internal/creelengine` imports neither `internal/reedengine` nor any package under `internal/shuttleengine/` and names no tmux command, so every keystroke goes through the `Delivery` seam its CLI implements over shuttle, checked by `TestCreelStore_DeliverySeamImports`.
It also states the layering: creel decides what and when, shuttle knows the provider, reed does keystrokes, and reed never imports creel.
`TestCreelStore_DeliverySeamImports` walks the production files of `internal/creelengine` with `scankit.Walk` (shallow, floor of one file) and fails naming the file on an import of `internal/reedengine` or of `internal/shuttleengine` or any path under it; its detector subtest feeds a synthetic file importing `internal/reedengine` and one importing `internal/shuttleengine/claudeengine` and asserts each is reported.
`pattern/PATTERN-told-geometry.md`'s Bound packages list gains `creelengine`.
`cmd/lyx/testdata/test-budget.yaml` raises `internal/creelengine` by one.

**ImpactSummary:** One new PATTERN entry with compensating wording cuts in other entry lines, a new background, one bound-package name and one new scan test; no code behavior changes.
