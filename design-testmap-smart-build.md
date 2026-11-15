# Study how to build and keep per-test coverage maps cheaply enough to pay off

`testmap-smart-build` · Task · research, testkit, gate, dobby · running · Describe

## Problem

The coverage maps (`internal/testcover`, built by `lym gate testmap`) record, per package, which code each top-level test covers.
They serve two uses: the gate picks the tests a change reaches, and `lym gate redundant` finds tests other tests already cover.
Today they are too slow to keep fresh:
the payoff report `docs/research/speed/testmap-payoff-2026-11-15.md` measured an upkeep pass after one landing at 7.7 to 31 minutes,
of which the tests the commit affects took 1.5 s and 21 s, and the gate saved 0 to 4.2 s.
Upkeep is switched off hub-wide (`test_map_upkeep: false`), and the gate is to stop using maps no upkeep keeps (note `testmap-on-command`).

The cost comes from how a map is built: the package's test binary is built with `-coverpkg` over every module package it links,
and each test runs alone in its own process, because Go's coverage counters are process-wide and parallel tests in one process mix their counts.
A change the selector cannot place, such as a type, signature or constant edit, sends the whole package back through that.

The operator holds that the maps can be built and kept on a smart, cheap footing and pay off; the current design is not optimal yet.

## Question

How can LoomYard build and keep per-test coverage maps cheaply enough to pay for themselves, for both uses?

## Directions to weigh (operator and orch, 2026-11-15)

Weigh at least these, and any better one the study finds:

1. **Record as tests run.** Every ordinary gate run logs which code each test ran or touched, so maps refresh as a side effect of tests that run anyway, with no separate upkeep pass.
   Settle how a run attributes code to one test when tests run in parallel and start goroutines.
2. **One process, snapshots between tests.** Run a package's tests in sequence in one process and use `runtime/coverage` to write and clear the counters between tests, instead of one process per test.
3. **Static reachability.** Compute from source which functions each test can reach, by the call-graph analysis the `PATTERN-test-only-code` guard already runs (`cmd/lym/testonlycode_linux_test.go`).
   It over-approximates, which is safe for selection, and may be too coarse for cutting redundant tests.
4. **Cheaper instrumentation.** Function-level instead of block-level coverage, a narrower `-coverpkg`, or build-cache reuse across packages.
5. **Combinations,** such as static reachability for gate selection and dynamic coverage on command for redundancy cuts.

For each: what it costs per landing and per full build, how fresh and how exact the maps stay, what it needs from Go and the toolchain, and what it changes in `internal/testcover`, `internal/gatecli` and the dobby.

## Bounds

- A study: read the code, the reports under `docs/research/speed/`, and Go's documentation.
- Small probes are allowed to check a claim: at most three, each on one package, each run once,
  such as timing one package's map built per process against one process with snapshots.
  Change no production code and write nothing under Bolt.
- The result is a report under `docs/research/speed/` that ends in one recommended design with its expected cost per landing and per full build,
  and the open risks; the decision is the operator's.
