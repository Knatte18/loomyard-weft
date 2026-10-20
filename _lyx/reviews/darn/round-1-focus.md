---
round: 1
focus:
  - "internal/gatecli/prebuild.go `storePrebuilt`: settle whether a concurrent prune between the existence check and `os.Chtimes` makes the run fail instead of republishing, whether the rename-race and prune paths can delete or replace an entry another running gate still uses, and whether the store path stays derived from `PrebuiltDir` with no new `.lyx` literal."
  - "internal/gatecli test path, inside and outside a hub: settle whether the tagged run exports a stable content-named path in a hub and keeps the per-invocation `-o` path and its cleanup outside one, and whether the stripped `LYX_TRACE_ID` reaches go test's environment at every spawn site, not only the one changed."
  - "internal/loomshed verify gate, round and whole-diff forms: settle whether a Publish failure record's failing tests run before the told or derived command in both forms, whether the `tmux` pass still follows, and whether a record with no failing tests or an unparsable one still falls through to the full command."
  - "Verify-tier change (`CLAUDE.md`, the loom plan's `## verify:`, `pattern/PATTERN-test-speed.md`, `internal/languageconfig` doc): settle whether every place that still names an untagged `go test` tier before the `integration` run was updated, and whether PATTERN and the docs land in this diff as CLAUDE.md requires for the changed behavior."
  - "`logger.TraceIDEnv` rename in `reedengine` and `shedverbs`: settle whether any literal `LYX_TRACE_ID` use remains in non-test or test code that should use the constant."
  - "Test economy for the new tests (`TestStorePrebuilt`, the tagged-run tests, the loomshed ordering tests): settle whether each new behavior has coverage, and whether any new test duplicates an existing one, naming the covering test."
  - "`_lyx/landing/summary.md` claims, including the benchmark figures in `docs/benchmarks/gate-test-xdr.md` and the `internal/shedbuild` fixture change: settle whether the diff bears out each claim."
---

Focus set from the board entry's three asks: verify runs the integration tier alone, a re-run on an unchanged tree reuses passed packages, and a failed test runs first.
