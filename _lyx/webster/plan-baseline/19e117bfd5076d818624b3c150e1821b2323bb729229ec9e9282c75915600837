# Card 4 — card-gate-lint-pattern

**Edit:**
- `internal/websterengine#cardGateCommand`
- `internal/websterengine#TestCardGateCommand`
- `internal/websterengine#TestRenderCardGates_OneBulletPerCardWithAStepSubBulletEach`
- `internal/websterengine_test#TestBeginBatch_PromptFile`
- `internal/websterengine/doc.go`
- `contracts/stencils/webster/webster-body-implementer.md`
- `contracts/stencils#wordingClaims`
- `pattern/PATTERN-gate-self-check-parity.md`
- `pattern/PATTERN-test-speed.md`
- `docs/overview.md`

**Intent:** Append `lyx loom lint-pattern` to every webster card gate after `lyx loom lint-comments`, the first of decision D3's two gate sites.
`cardGateCommand` appends the two lint steps in that order, so a card with no surviving package directory runs the two lints alone.
`TestCardGateCommand`'s expectations and `TestRenderCardGates_OneBulletPerCardWithAStepSubBulletEach`'s rendered text end with both steps, and `TestBeginBatch_PromptFile`'s card-gate expectation does the same.
`internal/websterengine/doc.go`'s sentence on a card's own gate names both lints.
The implementer stencil's always-allowed sentence names `lyx loom lint-pattern` beside `lyx loom lint-comments`, and the `wordingClaims` row that pins that sentence is updated to the new wording, so the claim test stays green.
`pattern/PATTERN-gate-self-check-parity.md` gains the pair of every card gate's headroom lint and `lyx loom lint-pattern`: `patternlint.Lint` over `pattern.Headroom`.
`pattern/PATTERN-test-speed.md`'s gate-per-tier table names the headroom lint in the card-gate row after the comment lint.
`docs/overview.md`'s line that every card gate ends with the comment lint says the gate ends with the two lints.
Bound: the card gate is run by the fork, not by Go, so a fork that skips the step lets its card's growth through until the Webster-Burler round gate of card 6 checks the whole branch's growth, and the hard cap in the plan-level verify bounds that.

**ImpactSummary:** One more step in the derived card gate; the three tests that pin the gate's text and the stencil claim row change with it.
