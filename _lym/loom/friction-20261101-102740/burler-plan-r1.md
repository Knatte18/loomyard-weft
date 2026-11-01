# Burler plan round 1 fix: a test file with no glyph cannot be targeted

`cmd/lym/seamsignature_test.go` pins each cli module's `RunCLI`/`RunCLIIn` through blank-identifier vars only, so `lym quarry toc` reports no symbols for it.
`validate-plan`'s `edit-test-file` rule refuses the file as an Edit target and asks for member glyphs, so a card adding a cli module cannot plan the pin entry, and the self-check finding asking for it had to be deferred.

Separately, inserting a card renumbered every card after it; `insert-card` rewrites only `<N>-<slug>` ids, so the bare "card N" mentions (about twenty) had to be renumbered by hand.
