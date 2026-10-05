# Measure and prune the test suite

`test-suite-prune` · Task · enhancement

Depends on: [`test-writing-rules`](design-test-writing-rules.md)

Stage 2 of getting the test suite under control: measure it, then cut it down.
It runs after `test-writing-rules`, so its own changes follow the new rules, and runs alone: it touches test files in nearly every package.

The suite grew from 1967 test functions on 2026-08-13 to 5749 on 2026-10-05, and the integration tier from 17.9 s to 57.5 s on the same idle machine.
The two test revisions of 2026-10-03 (`22e46970e`, `5805c58fb`) removed duplicated helpers and hardcoded expectations, but left the count unchanged.

## Measure

- Build a tool (e.g. `cmd/testprune`) that runs per-test coverage and lists tests with no unique coverage: every line they cover is covered by another test.
  It reports per package: test count, wall time, serial time, and the redundant candidates.
- Record the before-state in `docs/benchmarks/test-suite-timing.md` with its slowest-tests table.

## Clean

Package by package, largest first (`internal/fabricengine`, `internal/websterengine`, `internal/reedengine`, `internal/shuttleengine`, `internal/loomcli`, `internal/shedadapters`, then the rest):

- Delete the tests the tool reports as redundant, after checking each one does not pin a contract the coverage view misses (an error message, an output shape).
- Fold near-identical tests into table-driven tests.
- Move integration tests that need neither git, tmux nor processes to the untagged tier with `internal/testkit` fakes.

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

## Consolidate PATTERN's Testing section

`test-writing-rules` added three Testing entries (`PATTERN-test-economy`, `PATTERN-test-time-seam`, `PATTERN-test-parallel`); the operator wants the section revised, not grown.
Proposal to settle in Discussion: merge `test-tier-purity`, `test-time-seam` and `test-parallel` into one `PATTERN-test-speed`, and `hermetic-git-tests` and `tmux-test-isolation` into one `PATTERN-test-isolation`, leaving Testing with test-economy, test-speed, test-isolation, testkit and hubforge-fixtures.
Enforcement tests that name the old entries move with them.

## Align with the rewritten scribe testing skills

The general skills `scribe:testing` and `scribe:golang-testing` were rewritten to match `test-writing-rules` (scribe `cfad880`), and that run aligned its stencils with them.
While consolidating Testing (above), let `PATTERN-test-economy`'s background point at the skill for the general rules and keep only what is project-specific or stricter (e.g. a redundancy finding names the covering test).

## Budget guard

Once the suite is pruned, a guard test pins a per-package budget for test count and wall time at the after-state; exceeding it fails until the budget is raised deliberately in the same commit.

## Done when

- The after-state is recorded beside the before-state in the benchmark doc.
- The test count and the integration tier's wall time have dropped substantially; the task's PR states both numbers.
- Coverage of non-test code has not dropped beyond what deleted redundant tests explain.


