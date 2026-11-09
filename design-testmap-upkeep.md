# Decide how the coverage maps are kept fresh: triggers, batch, staleness and sharing across machines

`testmap-upkeep` · Note · enhancement, testkit, gate, dobby

## Seen (orch, 2026-11-09)

The coverage maps exist as code since project `redundancy-check` landed, but how they are kept fresh was settled piecemeal:

- The prime dobby starts `lym gate testmap --max <test_map_batch>` (default 5) after every `test_map_every` (default 5) landings, and nothing else starts one.
- A map is stale when its package's Go files at HEAD changed, when it is older than `test_map_max_age_days` (7), or when it records a failed build at another HEAD.
  A change to a dependency, or to a non-Go file of the package, does not mark it stale; only the age limit bounds that drift.
- A stale package's map is rebuilt whole: `internal/testcover` builds a coverage binary and runs every top-level test alone, serially, one process per test.
  Per-test coverage needs that, since coverage counters are process-wide and tests run in parallel in one process mix their counts,
  so an ordinary gate run of the package cannot update its map.
- The maps live under `_bolt/.lym/testcover`, which git excludes, so each machine builds and keeps its own set:
  a move to another machine, by `lym adopt` or otherwise, starts from no maps.
- A refresh holds a gate slot while it builds, so runs' gates wait behind it.

The operator finds keeping the maps fresh this way too large a job to run again and again.

## Proposal (orch)

1. Update a map in part instead of rebuilding it whole.
   A map keys blocks by content and carries each test's source hash, so after a change only these tests need to run again:
   a test that is new or whose source changed, and a test whose recorded blocks include a block that changed or was removed.
   A test that touched none of the changed code takes the same path as before.
   After a landing that is usually a handful of tests, not the package.
2. A stale or missing map never gives a wrong answer, only a slower gate:
   a gate that scopes by the maps runs the whole package when its map is missing or stale.
   That makes a lagging map harmless, so upkeep can run lazily.
3. Build when the machine is idle, with no gate waiting, rather than holding gate slots while runs work.
4. Drop or lengthen the weekly age rebuild: with item 2, dependency drift costs only scope, not correctness.
   A change in a package this package imports can still change a test's path without changing the package; decide whether that needs more than the age limit.
5. Share the maps across machines, for instance through the Bolt checkout or the records branch:
   they are keyed by content, not by machine, so each map is built once in all.
6. Cutting existing redundant tests (`redundancy-sweep-existing`) is a one-off that needs fresh maps only when it runs;
   only gate scoping (`gate-scope-by-coverage-map`) needs maps kept fresh.

Note `testmap-backlog-drain` is one part of the trigger question; this note owns the whole policy.

## Research first

Before the design, three agents brainstorm, each on its own and without seeing the others' answers,
whether there is a smarter way to keep per-test coverage maps fresh:
Fable, Codex astra, and Opus at high effort.
Each reads `internal/testcover`, `internal/gatecli/testmap.go`, the prime dobby's refresh step and this note, and writes its own proposal.
One brainstorm per agent, three in all.
The proposals land together in one report under `docs/research/`, compared side by side, as the input for the design.
Codex fails on OSL-1046 for its bubblewrap sandbox, so the research runs on Neve.
