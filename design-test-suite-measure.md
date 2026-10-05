# Measure the test suite, make it fast, settle its rules

`test-suite-measure` · Task · enhancement · running · Describe

Stage 2a of getting the test suite under control: measure it, make it fast, and settle the rules before pruning.
`test-suite-prune-core` and `test-suite-prune-rest` follow and delete tests using this task's tool and report.

The suite grew from 1967 test functions on 2026-08-13 to 5749 on 2026-10-05, and the integration tier from 17.9 s to 57.5 s on the same idle machine.
The two test revisions of 2026-10-03 (`22e46970e`, `5805c58fb`) removed duplicated helpers and hardcoded expectations, but left the count unchanged.

## Measure

- Build a tool (e.g. `cmd/testprune`) that runs per-test coverage and lists tests with no unique coverage: every line they cover is covered by another test.
  It reports per package: test count, wall time, serial time, and the redundant candidates.
- Record the before-state in `docs/benchmarks/test-suite-timing.md` with its slowest-tests table, and commit the tool's per-package candidate report where the prune tasks can read it.

## Speed

- Give real-time waits a test seam: the reed watchdog discovery interval (`watchdog.go`), `reapExitTimeout` (`reedengine/lifecycle.go`), verifyrun's `waitDelay` (`verifyrun/run.go`); use `true` instead of `sleep 5` in `verifytree/verify_integration_test.go`.
- Add `t.Parallel` to fabricengine's serial integration files where tests are independent (`mergein_recovery_integration_test.go`, `commit_integration_test.go`, `pull_integration_test.go`, `warpbinding_clone_integration_test.go` and others; 425 of 698 tests run serially).
- In `bareVocabularyTokenIn` (`lyxcwd/enforcement_test.go`), skip the regex when the lowercased text contains neither "weft" nor "warp".
- Arm the hermetic git env in `gitrepo/internal/gitoracle`.

## Socket leak (from note `tmux-socket-leak`)

Test runs leave dead tmux socket files in `/tmp/tmux-1000/`: 659 on 2026-10-05, and every suite run adds more.
Prefixes, most to least common: `lyx-warp-bare-LYXHUB-*` (over 400), `lyx-001-*` (standalone geometry, `internal/standalonegeom/reedgeom.go` `SocketKey`), `lyx-contract-*-test-<pid>-<n>` (`internal/reedengine/contract_integration_test.go`), `lyx-dotfill-harness-*`, `lyx-attach-harness-<pid>` (`internal/reedcli/smoke_attach_test.go`), `lyx-<hash8>`, `lyx-Test*`.

- `kill-server` stops a server but leaves its socket file: remove the socket in test cleanup through one shared helper (see `pattern/PATTERN-tmux-test-isolation.md`), and point `TMUX_TMPDIR` at `t.TempDir()` where possible so the files never reach `/tmp`.
- Check whether production reed paths (standalone geometry, warp-bare) also leave sockets.
- A full suite run leaves no new file in `/tmp/tmux-$UID/`.

## Test tiers (from note `test-tiers`)

Seen 2026-10-03: a Webster implementer fork ran `-tags smoke` and spawned about 24 real Claude Code sessions through the `orchcli` smoke tests (`TestSmokeOrch_*`).
The smoke tier mixes tests that drive only real tmux and the built binary with tests that spawn real `claude` sessions (orch cycle and adopt, shuttle runs, Burler clusters, the treadle judge, the model pin); the latter skip only when `claude` is not on `PATH`.

Decided (operator, 2026-10-03): four tiers, split by what a test needs and costs:

1. offline unit tests;
2. git operations (time, no tokens);
3. live tmux and the built binary (time, no tokens);
4. real LLM sessions (tokens), under its own build tag, run only when explicitly asked, never by a card verify, the plan verify or PR-Gate.

The fourth tier is named for LLMs, not Claude (e.g. `llm`): lyx is not meant to drive Claude only (operator, 2026-10-05).
A tier-3 test never spawns an LLM: it uses a fake or no provider.
Settle the other tier names in Discussion ("smoke" says nothing about tmux), retag the tests that spawn an LLM, and state the tier rules in PATTERN's Testing section (below) and in Plan-Write's per-card verify.
Which gate runs which tier stays in note `test-tiers`.

## Consolidate PATTERN's Testing section

`test-writing-rules` added three Testing entries (`PATTERN-test-economy`, `PATTERN-test-time-seam`, `PATTERN-test-parallel`); the operator wants the section revised, not grown.
Proposal to settle in Discussion: merge `test-tier-purity`, `test-time-seam` and `test-parallel` into one `PATTERN-test-speed` (which also carries the tier rules), and `hermetic-git-tests` and `tmux-test-isolation` into one `PATTERN-test-isolation`, leaving Testing with test-economy, test-speed, test-isolation, testkit and hubforge-fixtures.
Enforcement tests that name the old entries move with them.

The general skills `scribe:testing` and `scribe:golang-testing` were rewritten to match `test-writing-rules` (scribe `cfad880`).
Let `PATTERN-test-economy`'s background point at the skill for the general rules and keep only what is project-specific or stricter (e.g. a redundancy finding names the covering test).

## Done when

- The tool runs and its report and the before-state are committed.
- The integration tier's wall time has dropped from the speed fixes alone; the PR states before and after.
- A full suite run leaves no new socket file, and no tier below the LLM tier spawns an LLM.
- PATTERN's Testing section has fewer entries than before this task.
