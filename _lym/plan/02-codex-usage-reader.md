# Card 2 — codex-usage-reader

**Create:**
- `plan:internal/shuttleengine/codexengine#Codex.SessionUsage` -> `func (c *Codex) SessionUsage(sessionID, workdir string) shuttleengine.SessionUsage`
- `plan:internal/shuttleengine/codexengine#rolloutTotalUsage` -> `func rolloutTotalUsage(path string) (shuttleengine.SessionUsage, bool)`
- `plan:internal/shuttleengine/codexengine#TestSessionUsage` -> `func TestSessionUsage(t *testing.T)`

**Edit:**
- `internal/shuttleengine/codexengine#rolloutEvent`
- `internal/shuttleengine/codexengine#readRolloutEvents`
- `internal/shuttleengine/codexengine/doc.go`

**Uses:**
- `internal/shuttleengine/codexengine#tokenCountLine`
- `internal/shuttleengine/codexengine#writeRollout`
- `internal/shuttleengine/codexengine#sessionsDirName`
- `internal/shuttleengine#UsageReader`

**Intent:** D12's Codex reading.
`Codex` implements `shuttleengine.UsageReader`, with a `var _ shuttleengine.UsageReader = (*Codex)(nil)` assertion.
`SessionUsage` finds the session's rollout under the sessions directory of the Codex home the engine runs (`sessionsDirName`, which links to the operator home's), as the `rollout-*-<sessionID>.jsonl` file of that session, and reads it through `rolloutTotalUsage`.
`readRolloutEvents` and `rolloutEvent` also decode `payload.info.total_token_usage` of a `token_count` event: `input_tokens`, `cached_input_tokens`, `output_tokens` and `cache_write_input_tokens` when the total carries it.
`rolloutTotalUsage` takes the last `token_count` event carrying a total and returns: `Input` = `input_tokens` − `cached_input_tokens`, `CacheRead` = `cached_input_tokens`, `Output` = `output_tokens` (which already holds `reasoning_output_tokens`, never added again), `CacheWrite` = `cache_write_input_tokens`, or zero when the total carries no such key, never unknown; `SplitKnown` and `Known` true, `Fresh` the sum of input, cache-write and output.
A missing session file, an unreadable file or a rollout with no total returns the zero value, every count unknown, and no error.
`TestSessionUsage` (tier0, rollouts written by `writeRollout` with a new total-carrying line beside `tokenCountLine`) has rows: a total with `cache_write_input_tokens`, a total without it (cache-write zero), two totals where the last wins, a `reasoning_output_tokens` beside the output that does not change the output, a rollout with no total and a session with no rollout file (both zero value).
`codexengine/doc.go` replaces "The rollout's newest token_count gives the context size" with the context size and the session usage both read from the rollout's `token_count` events.
The provider seam holds: the split is read only in this package (`PATTERN-shuttle-provider-seam`); Codex still implements no `MessagePricer`, so its dollar cost stays unknown.

**ImpactSummary:** Widens one internal rollout decoder by a field read the context-size reader ignores; the new method is reached only through the `UsageReader` assertion in shuttle's finalize.
