---
round: 1
focus:
  - "internal/testkit/tiergate/tiergate.go: settle whether parsing of LYM_TEST_TIER (single tier, range such as 0-2, unset, malformed, reversed range) matches the record's design, and whether Require skips exactly the tests whose tier is outside the selection."
  - "internal/impactset/guards.go and the cmd/lym tier, purity and LLM guards: settle whether RequiredTier finds the Require call for Test, Benchmark and Fuzz functions alike, and whether the tag fallback for an older tree can misclassify a file or hide a violation."
  - "cmd/testtiming/main.go and spawnscan.go: settle whether setting LYM_TEST_TIER from -tags selects the tiers the old build tag selected, and whether the spawn scan finds the same spawns as before."
  - "internal/testkit/tmuxkit (allow_none.go, allow_tmux.go removed; tmuxallowed.go added): settle whether reading the tmux-server switch from the selection keeps the old behavior for tier1 and non-tier1 runs."
  - "The rewritten test files across the module: sample packages with a tier1 or tier2 TestMain, subtests, t.Parallel, helpers and package-level setup, and settle whether any test lost its Require call, carries the wrong tier number, or runs setup in TestMain regardless of the selected tier."
  - "The 13 removed !tier1 TestMain companions: settle whether each package's remaining TestMain still behaves correctly for every tier now that one binary serves all tiers."
  - "Scope against the record: settle whether the diff leaves gate.yaml untouched, writes no per-tag fallback file, and commits no .scratch rewrite program, as the amended decisions require, and whether PATTERN.md, pattern/PATTERN-test-speed.md, pattern/PATTERN-test-isolation.md, contracts/specs/loom-plan-spec.md and docs/benchmarks/running-tests.md describe the run-time selector without contradicting the code."
  - "The change description, if present, against the diff: settle whether its claims (counts excepted) about the tag removal, TestMain removal and static readers hold, and whether it states that the orch drops -tags and per-tag go vet after landing."
---

Seed focus only; each entry names where to look and the question to settle.
