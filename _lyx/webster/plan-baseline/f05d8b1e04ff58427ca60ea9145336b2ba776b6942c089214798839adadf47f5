# Card 2 — Failure episodes, the level policy text and PATTERN-log-levels

**Create:**
- `internal/logger/episode.go`
- `plan:internal/logger#Episode` -> `type Episode struct`
- `plan:internal/logger#Episode.Fail` -> `func (e *Episode) Fail(msg string, args ...any)`
- `plan:internal/logger#Episode.Recover` -> `func (e *Episode) Recover(msg string, args ...any)`
- `plan:internal/logger#EpisodeSet` -> `type EpisodeSet struct`
- `plan:internal/logger#EpisodeSet.Fail` -> `func (s *EpisodeSet) Fail(key, msg string, args ...any)`
- `plan:internal/logger#EpisodeSet.Recover` -> `func (s *EpisodeSet) Recover(key, msg string, args ...any)`
- `internal/logger/episode_test.go`
- `plan:internal/logger#TestEpisode_WarnsOncePerEpisodeAndInfoOnRecovery` -> `func TestEpisode_WarnsOncePerEpisodeAndInfoOnRecovery(t *testing.T)`

**Edit:**
- `internal/logger/logger.go`
- `PATTERN.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
D12, and the one mechanism every later card's repeating-failure guard uses (D3, D11).

`Episode` (zero value ready, one owner, no lock) tracks one failure that can repeat on every tick:
`Fail` counts the failure and logs `msg` with `args` at Warn on the first failure of an episode and at Debug, with a `failed_ticks` field carrying the count, on every later one;
`Recover` does nothing outside an episode, and inside one logs `msg` at Info with `failed_ticks` and ends the episode, so the next `Fail` is a Warn again.
`EpisodeSet` (zero value ready) is a mutex-guarded map of `Episode`s by key for a caller that tracks several failures, or one per strand or entry, from one value: its `Fail` and `Recover` act on the key's episode, and `Recover` on a key with no episode does nothing.
Both stamp nothing of their own beyond `failed_ticks`; the caller's message and fields are logged as given, so a caller keeps its existing message text and a test grepping it still finds it.

`logger.go`'s `# Level policy` section gains the three rules D12 names, after the existing Hard rule paragraph:
an ordinary state (a wait, a pause, a halt for the operator, an ordinary retry iteration) is Info at its change and never Warn;
a failure that repeats each tick logs Warn once per episode, Debug while it persists, and Info at recovery, which `Episode` and `EpisodeSet` implement;
a log line's dedupe key holds no text that changes while the state stays the same.

`PATTERN.md` gains, as the last entry under `## Daemons and waits`, this line with no `(test)` tag and no background link:

```
- `PATTERN-log-levels` — Logging in a loop or poll: follow `internal/logger`'s Level policy.
```

`PATTERN.md` stands 4 bytes under the 15564-byte headroom line `lyx loom lint-pattern` and the round verify gate enforce, and the entry adds 95, so the same commit shortens four entry lines whose detail their background files already carry, by 102 bytes together:
`PATTERN-config-strictness` becomes "Loading config: a caller adopts exactly one of strict `configengine.Load` or `LoadOrTemplate`.";
`PATTERN-glyph-conversion-chokepoint` becomes "Converting glyphs and paths: only through `glyph`'s converters, plus `planparser`'s one external-test unit mapping.";
`PATTERN-wait-mark-display-only` becomes "Marking a wait on screen: set the wait marks only through reed's `SetWaitMark`; no Go decision reads them.";
`PATTERN-config-layers` becomes "Resolving a module's config: only `configengine` reads, stacks and locates layer files; `configreg` sets keys closed to the local layer.".
Each keeps its tags and background link, and the card is done only when `lyx loom lint-pattern` reports no growth past the headroom line.

`TestEpisode_WarnsOncePerEpisodeAndInfoOnRecovery` is table-driven over a captured logger at Debug verbosity (`withCapturedOutput` plus `SetVerbosity(2)`, not parallel since both are process-global), one row per sequence of calls, asserting the exact level and `failed_ticks` of each line:
three `Fail`s then one `Recover` log one WARN, two DEBUG with `failed_ticks=2` and `failed_ticks=3`, and one INFO with `failed_ticks=3`;
a `Recover` with no failure logs nothing;
a `Fail` after a `Recover` logs WARN again;
on an `EpisodeSet`, two keys failing alternately each log their own first WARN, and recovering one key leaves the other's episode running.
`cmd/lyx/testdata/test-budget.yaml`'s `internal/logger` row rises from 36 to 37.

**ImpactSummary:** a new leaf type pair in `internal/logger` with no caller yet, package-doc text in `logger.go`, and four PATTERN.md entry lines; nothing existing changes behavior.

**Commit:** 2: logger: failure episodes, the level policy rules and PATTERN-log-levels
