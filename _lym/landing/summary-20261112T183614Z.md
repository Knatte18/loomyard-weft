# Coverage-map pilot on internal/adoptcli, and `lym gate redundant`

This change answers the pilot's three questions for one package, `internal/adoptcli`,
in `docs/research/speed/coverage-map-pilot-2026-11-12/report.md`.
It also adds the one verb the pilot lacked: a candidate list over a whole package's coverage map.

## What changes

- **`lym gate redundant <package>`** (`internal/gatecli/redundant.go`).
  It reads the package's coverage map from the hub and lists its redundancy candidates:
  each test whose covered blocks the package's other judged tests cover,
  with its tier, standalone seconds, covering tests and whether it is in the removable set.
  It lists keep-marked tests with their reasons, and tests not judged with why.
  It runs no test and takes no slot, and it works from any worktree of the hub.
  It refuses outside a hub, for a missing, unreadable or build-failure-only map, and for a failed keep scan,
  each refusal with a way forward.
- **`testcover.JudgeMap`** judges a stored map's entries with the existing package-scoped verdict.
  A test marked `//testtiming:keep` is neither judged nor a cover;
  an entry recording an outcome, or one the spawn scan marks, is not judged.
- **Report** with its scaling input `packages.tsv` and the package's candidate list.
- **Docs**: the `gatecli` and `testcover` package docs, `docs/overview.md`, and the coverage-map section of
  `pattern/PATTERN-test-speed.md`.

## What the report finds

- **Build**: one `lym gate testmap internal/adoptcli` succeeded in 34.3 s wall and 47.8 s CPU on one of three slots.
  Most of that is bookkeeping between rounds of tests, which the verb's `run_seconds` does not count.
- **Full set**: estimated at 45 to 165 minutes on one slot and 65 to 230 CPU-minutes.
  The upkeep pass had already built the full set today in 44 minutes, at the low end.
- **Cutting**: 2 of 30 judged tests are candidates, and both assert something their covers do not.
  The map is a reading filter for cuts, not a verdict.
- **Scoping**: over two landed commits the selector never dropped a covering test.
  It ran no test for one commit and fell back to the whole package for the other,
  on a one-line test-helper edit.

## Design decisions

- **The verb is new rather than a mode of an existing one.**
  The gate's `redundancy` report step judges only the tests a diff changes,
  and `cmd/testtiming -redundancy` measures afresh instead of reading the map.
  A read-only verb beside `testmap` is the smallest way to give the list the task asked for.
- **The judgment lives in `testcover`.**
  `JudgeMap` sits beside `Judge` so the verdict stays in one package and is tested at tier0;
  the verb's tier1 test covers only hub resolution, the answer's shape and its refusals.
- **The pilot ran although the full set already existed.**
  The prime's dobby began building every map with `36621f0e`, before this run started.
  The pilot ran as the board entry asks, and the report uses that pass's 44 minutes to check the estimate.
- **The scoping comparison re-bases the pilot map instead of building maps at old commits.**
  The task allows one build.
  A scratch program recomputed the map's file hashes at each commit's parent and ran `testcover.Selector` with the commit as target.
  The covering sets come from the same map at function granularity; the report states that bound.

## Testing

- `TestJudgeMap_KeptAndUnmeasuredEntriesNeitherJudgedNorCovering` (tier0) pins that a keep-marked test neither is judged nor covers,
  and that outcome and spawn entries are not judged.
- `TestGateRedundant` (tier1) runs the verb over a hubforge hub with a written map.
  It checks the answer's candidates, kept and not-judged lists,
  and the refusals outside a hub, for a package with no map, and for a map holding only a failed build.
