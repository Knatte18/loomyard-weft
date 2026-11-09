# Decide how the coverage maps are kept fresh: triggers, batch, staleness and sharing across machines

`testmap-upkeep` · Note · enhancement, testkit, gate, dobby

## Seen (orch, 2026-11-09)

The coverage maps exist as code since project `redundancy-check` landed, but how they are kept fresh was settled piecemeal:

- The prime dobby starts `lym gate testmap --max <test_map_batch>` (default 5) after every `test_map_every` (default 5) landings, and nothing else starts one.
- A map is stale when its package's Go files at HEAD changed, when it is older than `test_map_max_age_days` (7), or when it records a failed build at another HEAD.
  A change to a dependency, or to a non-Go file of the package, does not mark it stale; only the age limit bounds that drift.
- The maps live under `_bolt/.lym/testcover`, which git excludes, so each machine builds and keeps its own set:
  a move to another machine, by `lym adopt` or otherwise, starts from no maps.
- A refresh holds a gate slot while it builds, so runs' gates wait behind it.

## Idea

Decide, as one design, how the maps are kept:

1. What starts a refresh: landings, a prime HEAD move, a leftover backlog, an age sweep, or a mix.
2. How large a refresh is, against how long it may hold a gate slot.
3. Whether a dependency change should mark a map stale, and what the age limit is for.
4. Whether the maps stay per machine or are shared, for instance through the Bolt checkout or the records branch, so a second machine does not rebuild them all.
5. What a gate or report does when a package it needs has no map or a stale one.

Note `testmap-backlog-drain` is one part of item 1; this note owns the whole policy.
