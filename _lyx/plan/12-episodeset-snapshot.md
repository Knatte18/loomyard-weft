# Card 12 — episodeset-snapshot

**Create:**
- `internal/logger#EpisodeSet.Ticks`
- `internal/logger#EpisodeSet.Restore`
- `internal/logger#TestEpisodeSet_RestoreContinuesTheEpisode`

**Intent:**
Let a failure-episode set be saved and restored, so a seam that keeps failing across batten's short passes warns once per episode and logs one recovery (D4's `failures` inventory item).
`Ticks` returns a copy of each open episode's failed-tick count by key; `Restore` replaces the set's episodes with those counts, so the next `Fail` of a restored key logs at Debug and its `Recover` logs at Info.
`TestEpisodeSet_RestoreContinuesTheEpisode` (tier0, logs captured through `logger.SetOutput`, so not parallel, naming that process-global): a key failed once, saved and restored into a fresh set, fails again at Debug with `failed_ticks=2` and recovers at Info; a key absent from the restore warns on its first failure.
