# Card 2 — delete-before-reference

**Create:**
- `internal/planglyph/deleteorder.go`
- `plan:internal/planglyph#LaterDeleteReferences` -> `func LaterDeleteReferences(plan *planparser.Plan, deleting, later []planparser.Card, worktreeRoot string) ([]Finding, error)`
- `internal/planglyph/deleteorder_test.go`

**Edit:**
- `internal/planglyph/planglyph.go`
- `internal/planglyph/doc.go`
- `contracts/specs/loom-plan-spec.md`
- `internal/webstercli/cli_test.go`

**Intent:**
Add decision D2's blocking resolve-backed check, `delete-before-reference`, which refuses a plan that deletes a symbol before a later card has removed its reference to it (the `pair-lifecycle` and #401 shape).

`LaterDeleteReferences`, in the new `deleteorder.go`, takes the cards on the deleting side and the candidate later cards separately, so card 7's record-batch and recover-batch can call it with a begun batch's own cards against the unbegun ones.
For every **Delete:** target of a card N in `deleting` that is a member glyph or a package self glyph, it searches the code of every card in `later` numbered above N under its **Edit:** group: a member glyph's resolved span (the `Symbol.File` lines `Start`..`End` of quarry's found answer), or the whole file for a file path (plain or file self glyph).
Other Edit target shapes, code no later card edits, and a Delete target that is a file path are not checked.
A reference is the deleted identifier as a whole word, qualified as Go would spell it from the searched file's package: bare or as a selector (`.Name`) when the file sits in the deleted symbol's own package, package-qualified (`pkg.Name`, `pkg` being the deleted symbol's package name) from any other package.
For a method or field glyph (`Type.Name`) the identifier is the last name, reached as `.Name`.
For a package self glyph the reference is an import of the package's path.
A member Delete target that no longer resolves found (already deleted) has nothing to look up and is skipped.
Each match is one blocking finding attributed to the deleting card; its detail names the deleting card, the later card, the target and the file and line of the reference, and tells the writer to move the delete to a card after the later one.
Resolution goes through this package's batched `resolveTargets`; an infrastructure error returns wrapped in `ErrQuarryUnavailable`, as every other pass does.
Under a non-glyph `language:` it returns no findings and opens no repository.

Wire it into `resolvePass` in `planglyph.go`, after the canonicalization reload, with `current` (the pending cards) as both `deleting` and `later`, so `ValidateFormat`, `Validate`, `ValidateRework` and `ValidateDispatch` all run it, and at dispatch both sides are scoped to unbegun cards.

Bound, stated in the `LaterDeleteReferences` doc comment: the match is textual, so a same-named unrelated symbol inside a later card's span is also flagged; the fix it names (move the delete later) is always available, which is what makes a blocking false positive acceptable, and it never fires on code outside a later card's **Edit:** targets.

Docs: add `delete-before-reference` (blocking) to `doc.go`'s resolve-backed check list.
In `contracts/specs/loom-plan-spec.md`, add a row for `delete-before-reference` at the end of the "Validation checks" list, after `commit-subject-mismatch` and numbered next, so the list stays the one row per emitted ID the decision record asks for.
The row says the check is emitted by `internal/planglyph`'s resolve pass, not by `internal/planparser`, at `ValidateFormat`, `Validate`, `ValidateRework` and `ValidateDispatch`, with its rule in one sentence, and the list's intro sentence on the two planparser entry points gets a clause for the rows `internal/planglyph` emits.

Tests:
`deleteorder_test.go` drives `LaterDeleteReferences` over a small fixture repository (the shape `resolve_test.go` builds), table-driven: a card N deleting a function that card N+1's Edit file still calls is refused naming both cards, the target and the file:line; the same plan with the deleting card numbered after the editing card (the `pair-lifecycle` order) passes; a same-package bare reference and a cross-package `pkg.Name` reference both match, an unqualified same name in another package does not; a method target matches `.Name`; a reference outside a later card's member-glyph span does not match; a file-path Delete target is not checked.
In `internal/webstercli/cli_test.go`, add rows to card 1's table-driven `TestValidateCmd_PlanGateChecks`: a plan whose card 1 deletes a function card 2 still calls exits 1 with `delete-before-reference` under card 1's key; with the two cards swapped it exits 0.

**ImpactSummary:** Adds one resolve-backed pass to planglyph's shared resolvePass, so every plan gate and webster entry point gains one blocking check; no existing check changes.
