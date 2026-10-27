<!-- This is the over-long-guard-step line of webster's card-gate list in a fork or recovery prompt.
     guardNotice (guardstep.go) reads it through stencilstore, fills its two markers, strips this banner and trailing newline,
     and appends the text to the card gate list when a card's guard step would exceed the command limit.
     guard_length is the longest step's length in characters, and guard_limit is the command limit in characters.
lyx-stencil: sha256=03bd0459967824015fb5de4866bf23b1cdc4790e9601b9b6628724ec93412da0 build=d4824a3d68de466ba3db5015cf6212f7d41126f7 time=2026-10-14T16:27:00Z -->

The guard step would be {{.guard_length}} characters, over the {{.guard_limit}}-character command limit, so the card gates that would carry it run without it.
The plan-level verify still runs the guard tests.
