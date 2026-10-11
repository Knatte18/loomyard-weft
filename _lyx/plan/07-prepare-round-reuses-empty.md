# Card 7 — prepare-round-reuses-empty

**Edit:**
- `internal/parentreview#Store.PrepareRound`
- `internal/parentreview#TestPrepareRound`
- `internal/shedrecipe#discussionWriteEntry`
- `internal/shedrecipe#discussionSeatsEntry`

**Prosa:**
- `contracts/specs/shed-recipe-spec.md`

**Intent:**
D9: a fresh spawn reuses a latest round whose directory holds no request.
A run with no reviewer, or a spawn that died before its gate opened a request, left an empty `round-N/` behind, and every later fresh spawn added another.

`Store.PrepareRound` returns the latest round unchanged when it exists and its `Request` is nil, beside the existing reject and superseding-approve cases; otherwise it calls `BeginRound` as before.
Its doc comment names the third case.

`TestPrepareRound` gains a row for a latest round begun with no request, which `PrepareRound` returns as round 1.
Its "no round" row calls `PrepareRound` twice and asserts both calls return round 1 and `Root` holds one round directory, the two-fresh-spawns-with-no-reviewer case.
The "reject kept" row still returns the reject's round.

`discussionWriteEntry`'s and `discussionSeatsEntry`'s docs, and `shed-recipe-spec.md`'s `parent-review` paragraph, say a fresh spawn opens the next round unless the latest one holds no request, a reject or a superseding approve.

**ImpactSummary:** one more keep-the-round case in PrepareRound, read by both Discussion-Write producers' fresh-spawn preparation; a round holding a request behaves as before.
