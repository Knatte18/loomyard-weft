# Card 1 — tokencount: a `-no-cutoff` flag drops both archive-tag lower bounds

**Edit:**
- `tools/tokencount#run`
- `tools/tokencount#TestRunReadsArchivedShedStatus`

**Uses:**
- `tools/tokencount#runBound`
- `tools/tokencount#readShedRecord`

**Intent:** Carries D1.
`run` gains a bool flag `no-cutoff`, default false, with a usage string saying it turns off the archive-tag cutoff so every transcript in each slug's project directory counts.
With the flag set, each slug is treated as one with no older archive tag: the bound `run` passes to `CountRun` is the zero time in place of `runBound(finished[slug], running)`, and a non-nil record from `readShedRecord` has its `Older` set to the zero time before `r.useShedRecord(*record)`.
Both reuse the existing no-such-tag path, so `CountRun` and `useShedRecord` do not change.
With the flag set, the report header gains one line after the counted-runs line and before the skipped note, exactly `The archive-tag cutoff is off: every transcript in each slug's project directory counts.\n`; without it the header is unchanged.

The package doc comment at the top of `tools/tokencount/main.go` changes in the same commit.
The sentence ending "and no flag turns the filter off" becomes "The run's section reports how many were left out and the bound's date." and is followed by the flag's description: `-no-cutoff` drops the bound for every run, so a slug's earlier runs that share its project directory feed every figure the bound gates (tokens, cost, working intervals, the session count, the Webster forks, the Merriam starts, the begin-batch base, the transcript window and with it the orch charge), and it drops the next-older tag's clamp on the span start.
The description says that with `-calibrate` the base and Merriam rows may come from an earlier run, that the plan, read from the newest plan commit before the run's first Webster fork, may be an earlier run's, and with it the peak table, the start table's base and the identity card counts, or that the run may be listed with "no plan commit before the first fork", and that a span start taken from the earliest transcript when the seed has no started_at may be an earlier run's.
The sentence "one with no such line is left out when a bound exists" is replaced so it says such a transcript is left out when a bound exists and counted under `-no-cutoff`.
The span paragraph's "That start is bounded below by the commit date of the slug's next-older archive tag" sentence gains "unless -no-cutoff is set".
New comment lines are one sentence per line.

Bound: the flag is opt-in per invocation, defaults off, and changes nothing but the report it writes; without it every figure is as today.

Test, extending `TestRunReadsArchivedShedStatus` in `tools/tokencount/time_integration_test.go` (integration tier, the lowest that reaches `run` over real archive tags):
- The first `run` call's report asserts the cutoff-off line is absent; its existing left-out assertions for the finished and in-flight runs stay, pinning the unchanged default.
- A new slug `early` is added to the `older` and `newest` commits: at `newest` a seed `{"recipe":"loom","driver":"go","started_at":"2026-01-01T08:30:00Z"}` and a status with `Plan` at `2026-01-01T10:03:00Z` and `Publish` at `2026-01-01T10:12:00Z`; `older` gets no `early` file and only carries a tag; tags `archive/early/<sha>` on both commits, so its bound and `Older` are the older tag's 09:00.
- After the default and empty-hub runs at the end of the test (so `early`'s sessions cannot change the default run's "most recent" order or its skipped header), write `early`'s transcripts: `a.jsonl` with the burler title and user/assistant lines at 10:00–10:10 (the existing `userAt`/`stamped` shape), and `none.jsonl` with the burler title and one untimestamped `assistant(...)` line.
- Run `run` with `-no-cutoff` over `done`, `live` and `early`, writing to a fresh `-out` file, and assert: the header holds the cutoff-off line; no section holds "left out"; the `done`, `live` and `early` sections each hold `| burler | 2 |` (the earlier transcript, and for `early` the untimestamped one, counted); and `early`'s Producers table holds `| Plan | 1 | 1h33m0s |`, the span starting at the seed's 08:30 and not clamped to 09:00, which would give `1h3m0s`.
The without-flag counterpart of the untimestamped rule, left out under a bound, stays pinned by the existing bounded `CountRun` case in `count_test.go`.
The file's header comment gains that the test also checks `-no-cutoff`.

**ImpactSummary:** One flag local to `run` in a standalone tool; `runBound`, `CountRun`, `readShedRecord` and `useShedRecord` are unchanged, the launchers pass flags through, and no other package imports `tools/tokencount`.
