# Fixer report — discussion round 1

Every finding's premise held against the code; all 16 are fixed in `_lym/discussion/decision-record.md`.

## Changes

- **F1** — D4's resolution sentence now says `planindex.Index` gains a method resolving a list of glyphs against a worktree root, implemented in `internal/planglyph` with the `glyph-not-found` policy lifted out of its per-card form; the existing `ErrQuarryUnavailable` sentence stays.
- **F2** — D5's scan trigger names the `NumField` method on a `reflect.Type` or `reflect.Value`, or `reflect.VisibleFields`, directly or through a same-package `_test.go` helper or `envkit.NilSeams`.
- **F3** — fixed differently: D8 states how a target is classed as test code (a `_test.go` file target, a `_test` pseudo-unit, a member glyph D4's resolving method reports declared only in `_test.go` files).
  The proposed "resolved through the index the plan gate already holds" was kept in substance, but the self-check showed a Create glyph for a new test function cannot resolve yet, so D8 also classes an unresolved Create member glyph named `Test…`, `Benchmark…`, `Fuzz…` or `Example…` as test code, and D4 now says its method also reports declaring files.
- **F4** — D6's and D8's Bounds name the Uses escape hatch and what catches it.
- **F5** — fixed differently: D9 states the tier tags are a value read from `gate.yaml` as `RefuseRaw` is, the seam markers a reader over the worktree root called at check time through the `impactset` parser, and that both builders fill them the same way.
  The index is not added to `PlanContext`: `planContextFindings` gains the index its callers already pass beside `pc` (`internal/loomcli/validate.go:285`), so it is not held twice.
- **F6** — D2 says both callers run D1 and D2 even when `Validate` finds defects, appended after its findings; the darn history check stays clean-only.
  Self-check added a carve-out: D1 and D2 add nothing on a record `Validate` reports missing, and D2 nothing on a record missing `## Acceptance criteria`.
- **F7** — D1's two findings name ways forward; after the self-check, `decision-heading-malformed` also offers demoting a non-decision H3, and `decision-id-duplicate` says to update every mention of the id.
- **F8** — fixed differently: D3's root-entry rule now reads "an entry git tracks at the worktree root", and D3's Bound says a token under a root entry git does not track is untested.
  The proposed `.git` exclusion was not added: git never tracks `.git`, so it would be a no-op.
- **F9** — D3 excludes a token with a `...` segment, and the Bound lists `internal/...` as untested.
- **F10** — D5's NilSeams arm is spelled `NilSeams` inside package `envkit`.
- **F11** — D7 states the tier tags are the `gate.yaml` tier keys, which on a Go hub are the `-tags` build tags as `PATTERN-test-speed` requires; after the self-check, D7's Bound states the check assumes keys equal tags and what happens on a hub where they differ.
- **F12** — D10: the Discussion writer and Plan writer stencils gain the pointer, the chair gets it through its include; the D10 acceptance bullet matches.
- **F13** — the D5 acceptance bullet covers the non-struct marker glyph and the unmarked `envkit.NilSeams` call.
- **F14** — D3's Bound names the `<` placeholder exemption.
- **F15** — Auto-mode assumptions gains the D6 widening bullet.
- **F16** — D10's first sentence splits D1 and D2 (loom and darn) from D3 and D4 (loom only).

## Self-check

- fix-consistency, LOW — D3 Bound still says "root entry the worktree lacks": fixed as proposed (Bound now says "git does not track", with `.scratch/` as an example).
- fix-consistency, LOW — D8 acceptance bullet lacks the test-member-glyph case: fixed as proposed.
- fix-way-forward, LOW — `decision-id-duplicate` way forward misses non-acceptance mentions: fixed as proposed; duplicate of fix-proposal item 5.
- fix-way-forward, LOW — `decision-heading-malformed` way forward mints a fake decision for a sub-heading: fixed as proposed.
- fix-claims, LOW — the index in `PlanContext` would duplicate the validators' index argument: fixed as proposed.
- fix-proposal, MEDIUM — F6's unconditional run cascades on a missing record or section: fixed as proposed (carve-out in D2).
- fix-proposal, MEDIUM — F3 misclasses an unresolvable Create test glyph, and no index method reports declaring files: fixed with my own change (the test-name prefix rule for unresolved Create glyphs, and D4's method reports declaring files).
- fix-proposal, MEDIUM — F11 asserts the key/tag convention as fact: fixed with the Bound option the lens offered, not by reading `-tags` values, since `PATTERN-test-speed` already requires the match.
- fix-proposal, LOW — D3 Bound stale and the `.git` exclusion is a no-op: fixed as proposed; the Bound half duplicates fix-consistency LOW 1.
- fix-proposal, LOW — renumbering falsifies other references: duplicate, fixed above.

## Disputed

None.

## Deferred

None.
