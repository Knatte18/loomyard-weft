# Card 3 — claude-pricing

**Create:**
- `plan:internal/shuttleengine/claudeengine#modelPrice` -> `type modelPrice struct`
- `plan:internal/shuttleengine/claudeengine#prices` -> `var prices map[string]modelPrice`
- `plan:internal/shuttleengine/claudeengine#Claude.PriceMessage` -> `func (c *Claude) PriceMessage(model string, u shuttleengine.MessageUsage) (float64, bool)`
- `plan:internal/shuttleengine/claudeengine#TestPriceMessage` -> `func TestPriceMessage(t *testing.T)`

**Edit:**
- `internal/shuttleengine/claudeengine/doc.go`

**Uses:**
- `internal/shuttleengine#MessagePricer`
- `internal/shuttleengine#MessageUsage`

**Intent:** `claudeengine` implements `shuttleengine.MessagePricer` (D4) in a new file `pricing.go`, with `var _ shuttleengine.MessagePricer = (*Claude)(nil)`.
`modelPrice` and `prices` are copied from `tools/tokencount/cost.go` with their doc comments, the price table's source citation, the Haiku 5.5 long-prompt card and the one-hour/five-minute cache-write split unchanged; card 5 deletes tokencount's copy.

`PriceMessage` is `messageCost`'s pricing over `MessageUsage`: a message whose every figure is zero returns `0, true` whatever its model, so a placeholder message is never unpriced; a model the table lacks returns `0, false`; a model with a long-prompt card is priced on it when `Input + CacheWrite + CacheRead` exceeds `LongPromptAbove`; the five-minute writes are `CacheWrite - CacheWriteOneHour`, each lifetime at its own rate, and the figure is in USD.

`TestPriceMessage` takes over `TestEstimatedCost`'s pricing rows, which card 5 drops from tokencount: one-hour and five-minute cache writes priced separately on Opus 5.5; Haiku 5.5 at exactly 100K prompt tokens on the lower card and one token over on the long card; an unknown model unpriced; a zero-usage message on a placeholder model priced at zero.

`doc.go` gains a sentence that the price table is Claude's API list price, with its source cited at the table.

**ImpactSummary:** additive: a new capability method and table in `claudeengine` and one doc sentence; tokencount's own table still serves its callers until card 5.

**Commit:** 3: claudeengine prices a message from its model and neutral usage
