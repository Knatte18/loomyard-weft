# Research how the coverage maps are kept fresh: three independent brainstorms

`testmap-upkeep` · Task · research, testkit, gate, dobby · running · PR-Review

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

## Starting ideas (orch)

These are ideas to weigh, not requirements; an agent may reject any of them.

1. Update a map in part instead of rebuilding it whole.
   A map keys blocks by content and carries each test's source hash, so after a change only these tests need to run again:
   a test that is new or whose source changed, and a test whose recorded blocks include a block that changed or was removed.
2. A stale or missing map never gives a wrong answer, only a slower gate:
   a gate that scopes by the maps runs the whole package when its map is missing or stale.
   That makes a lagging map harmless, so upkeep can run lazily.
3. Build when the machine is idle, with no gate waiting, rather than holding gate slots while runs work.
4. Drop or lengthen the weekly age rebuild, and decide whether a change in an imported package needs more than the age limit.
5. Share the maps across machines, for instance through the Bolt checkout or the records branch, since they are keyed by content, not by machine.
6. Only gate scoping (`gate-scope-by-coverage-map`) needs maps kept fresh; cutting existing redundant tests (`redundancy-sweep-existing`) needs fresh maps only when it runs.

Note `testmap-backlog-drain` is one part of the trigger question; this note owns the whole policy.

## This task: research only

The task builds nothing and decides no design.
It asks whether there is a smarter way to keep per-test coverage maps fresh, and it runs exactly three brainstorms, each by its own agent, none seeing another's answer:

- Fable 5.2 at high effort, twice, as two independent agents.
- Opus 5.5 at high effort, once.
- No Codex agent: Codex does not run on OSL-1046.

Each agent reads `internal/testcover`, `internal/gatecli/testmap.go`, the prime dobby's refresh step and this note, and writes its own proposal.
An agent only reads and writes its proposal; it runs no test suite and starts no further agent.
The proposals land together in one report under `docs/research/`, compared side by side, as the input for a later design task.
