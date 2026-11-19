```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens. For each thing the diff changes, I looked for every caller, doc, test and rule that reads it, and checked whether the diff leaves any of them wrong.

**Affected set per change**

- **`boardengine.ValidateEntrySlug` (new, exported) and `NewTask` now calling it**
  - Callers: `NewTask` (`task.go:430`) and the raddle context closure (`raddlecontext.go:86`). `NewTask` is reached from `store.go:602`, `:994` and `:1175`, which keep the same rule order and error text. Covered.
  - Sibling sites that still call the private helpers directly: `rename.go:73-76` and the patch path at `task.go:517-524`. Their behaviour is unchanged, and the doc's "every new slug passes" still holds for both. Covered.
  - Doc: `boardengine/doc.go` "# Slug rule" gains the new symbol. Covered.
- **`raddleparser.BoardContext.ValidateSlug` (new field)**
  - Builders: `loomcli/raddlecontext.go` sets it. `loomshed/raddlefile.go:481` (the in-write re-check) and `testkit/envkit/envkit.go:372` (a stub) leave it nil, and nil checks nothing. `RecheckBoard` never reads it. Covered.
  - Docs: the field's doc in `check.go:98-99` and `raddleparser/doc.go` are both updated. Covered.
- **New finding `slug-invalid` (`CheckSlugInvalid`)**
  - Spec check list: `raddle-proposal-spec.md:110`. Covered.
  - Spec rule paragraph at `raddle-proposal-spec.md:65` ("Every filed slug … is new: free on the board and in the raddle store, and not the Sketch's"): not updated. See finding 2.
  - Lists of check names elsewhere: none. A grep for `slug-duplicate` and `CheckSlugDuplicate` finds no other enumeration, test or stencil. Covered.
  - Raddle stencils: `contracts/stencils/raddle/` carries no slug-rule guidance, so nothing there goes stale. Covered.
  - Way-forward text: the detail carries a `way forward:` clause and names no denied step. `TestCheck_DetailNamesTheWayForward` asserts it. This satisfies `PATTERN-refusal-way-forward`. Covered.
- **The context's `ValidateSlug` closure (`ValidateEntrySlug`, then `fabricengine.ValidatePairSlug`)**
  - Its doc comment (`raddlecontext.go:33-34`), the spec line (`:110`) and `boardcli/doc.go:48` describe it.
  - The pair-rule half of "a board write refuses" is not true of Raddle-File's own board write. See finding 1.
- **Test rename `TestCheck_DependsOnNoteDetailNamesTheWayForward` → `TestCheck_DetailNamesTheWayForward`**
  - The old name survives only in the measurement data under `docs/research/speed/*`. Those are point-in-time reports, so they are exempt. Covered.
- **`PATTERN-leaf-packages`**: `raddleparser`'s imports are unchanged. The rule arrives as a function value, and the fixture adds only the stdlib `fmt`. Covered.
- **Commit messages and departures**: the `Root-Cause:` trailer on `87ce4100` names the defect: the checks carried none of the board's slug rules. The diff removes it. No `Pattern-Departure:` or `Loosened-Check:` trailer appears, and none is needed: the folded test keeps every depends-note assertion from the base. Covered.
- **Declared PATTERN changes**: none are declared. Nothing to check.

**Findings**

1. **LOW · consistency** · `internal/loomcli/raddlecontext.go:33-34`, and the same claim in `contracts/specs/raddle-proposal-spec.md:110`
   - Summary: both texts say the context's slug rules, the pair slug rule included, are "the rules a board write refuses a new slug by" / "a filed slug a board write would refuse". The write that files a raddle proposal does not apply the pair rule.
   - Evidence:
     - Raddle-File writes through `openHubBoard` → `boardengine.OpenHub` (`internal/loomcli/raddlewiring.go:101-107`, `internal/boardengine/config.go:256-265`), and its `Config` never sets `ValidateSlug`.
     - Only `lym board`'s pre-run sets it (`internal/boardcli/cli.go:164-170`).
   - Fix: reword both texts to say the pair slug rule is the one `lym board` writes and pair creation apply, not "a board write".
2. **NIT · consistency** · `contracts/specs/raddle-proposal-spec.md:65`
   - Summary: the spec's paragraph stating what a filed slug must be still lists only "new: free on the board and in the raddle store, and not the Sketch's". It omits the new rule that the board must admit the slug, so the rules section and the check list disagree.
   - Evidence: `raddle-proposal-spec.md:65-66` against `:110`.
   - Fix: add "and admitted by the board's slug rules" to that sentence.

**Tool calls**

- Grep for other lists of slug check names and for the old test name: my context did not show whether any doc, test or stencil enumerates the checks or names the renamed test.
- Grep of the raddle stencils and plugins for slug guidance, plus a read of `boardcli/doc.go:40-60` and `boardengine/config.go:50-58`: to see whether any stencil or doc describing slug rules goes stale.
- Grep of `contracts/stencils/raddle/` and `contracts/specs` for slug length or limit wording: to find prose stating the slug limit that might now disagree.
- Grep of the slug lines in `raddle-proposal-spec.md`: my context held only lines 90-125 of the spec, not its slug-rule paragraph.
