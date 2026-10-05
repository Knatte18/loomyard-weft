# Fixer report — plan round 1

All four findings fixed; the `approved:` key in `00-overview.md` was not touched.

## Changed

- F1: `15-editor-fallback.md` now says `configengine`'s `doc.go` gains the order as new text on its `[Edit]` line, and names the three existing grep matches.
- F2: `16-config-menu.md` states that the `menu` `RunE` is hand-written, checks `clihelp.ShouldAbort` first and records its exit with `clihelp.SetExit`.
- F3: `16-config-menu.md`'s last test bullet now asserts exit 1 and a JSON error envelope naming `bogus`, with no editor claim.
- F4: `15-editor-fallback.md`'s ImpactSummary now states the single caller, `runConfig`, and who sees the new order.

## Deferred

None.
