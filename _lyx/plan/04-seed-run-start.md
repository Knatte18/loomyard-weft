# Card 4 — seed-run-start

**Create:**
- `internal/loomcli#seedRunStart`
- `internal/loomcli#TestSeedRunStart`

**Intent:** D1: `internal/loomcli` reads the run's start, the lower bound of the landing's pull-request lookup, from the `started_at` stamp of the pair's own run seed, through `shedrun.ReadSeed(l, shedrun.SelfRunID)` only, per `PATTERN-shed-run-directory`.
`seedRunStart` goes in a new file `internal/loomcli/runstart.go`, with a file header naming what it reads, and not in `landingdeps.go`, whose header promises a file that does no I/O; later cards call it from the landing wiring and from the approve and reject verbs.

It returns:
- the stamp parsed with `time.Parse(time.RFC3339, …)` and a nil error, for a seed carrying a parseable stamp;
- the zero time and a nil error, with one `logger.Warn` saying the run's seed carries no usable start stamp so the landing's pull-request lookup is not bounded by the run's start, for a missing seed (`found` false), a seed with an empty `StartedAt` and a stamp that does not parse; the warning carries the seed's run id and, for an unparseable stamp, the stamp and the parse error;
- the zero time and `ReadSeed`'s error wrapped as `loom: read the run's start from its seed %s: %w`, the `%s` being `shedrun.SeedFile(l, shedrun.SelfRunID)` because `ReadSeed`'s decode error names no path, for a seed that exists but cannot be read or decoded, so the caller fails rather than silently dropping the bound.

The zero time filters nothing, so a zero bound keeps today's lookup (D1's bound); only a seed from before the `started_at` field, a removed seed or a hand-edited stamp reaches it.

`TestSeedRunStart` is one table over a location `&lyxcwd.Location{HubPath: t.TempDir(), WorktreeName: "pair", AnchorRel: "."}`, each row writing `shedrun.SeedFile(l, shedrun.SelfRunID)` itself after creating `shedrun.RunDir(l, shedrun.SelfRunID)`:
a seed with `"started_at": "2026-10-01T08:00:00Z"` returns that time and logs no warning;
no seed file, a seed with no `started_at` key, and a seed with `"started_at": "yesterday"` each return the zero time, a nil error and exactly one trace line carrying the warning's own `msg=` text;
a seed file holding `{not json` returns the zero time and an error carrying the seed file's path.
Each row calls `logger.SetDurableSinkDir(t.TempDir())` itself and registers `t.Cleanup(func() { logger.SetDurableSinkDir("") })`, so its trace holds only its own lines, and reads them from `logger.TraceFile()`, matching on the `msg=` text as `TestPublish_NoExistingPR_CreatesAndLogsOnce` does.
It never calls `t.Parallel`: the durable sink is process-global state, named in the test's comment.

**Commit:** 4: Read the run's start bound from the pair's seed in loomcli
