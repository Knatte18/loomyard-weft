# Writing card 7's producer table

The card required rendering the loom driver prompt through `loomcli.RenderDriverPrompt` with no production change,
but importing `loomcli` makes `internal/shuttleparity`'s test binary link tree-sitter,
which the tree-sitter link guard and `PATTERN-test-speed` forbid.
Only the card gate showed it, and it took a round trip to the parent to settle.
