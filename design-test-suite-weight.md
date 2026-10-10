# Lighter test suite: measured tiers, cheap tmux fixtures, no lingering children

`test-suite-weight` · Task · bug, enhancement, shuttle, reed, fabric · running · Webster-Bouncer

Depends on: [`gate-test-scope`](design-gate-test-scope.md), [`shuttle-send`](design-shuttle-send.md)

## Seen

On 2026-10-08 the integration tier dominated CPU during a Publish verify (`fabricengine.test` alone at 145%), and in the 53 hours before the crash tmux started 9,022 servers and 32,375 panes, nearly all from tier 3 tests, each pane a login shell that sources the operator's profile.
Tests in four packages link the `lyx` binary with cgo on their own, and `TestLoomResumeReconcilesTheHubConfigBeforeArming` flakes on `t.TempDir` cleanup because a git child, most likely a detached auto-gc, still writes into the fixture's object store (#472).

## Goal

- A measurement of every tier per package (time, CPU, peak memory, process count) with `cmd/testtiming`, on a quiet machine, before and after.
- Tier 3 panes run a non-login shell, and tests share one tmux server per package where isolation allows.
- `lyx` is built once per test run and reused across packages.
- Git fixture repositories are reused instead of built per test (operator, 2026-10-10): today most tests build their own repos and hubs through `gitkit` (about 1,400 calls, each spawning git) and tear them down again.
  A fixture shape is built once per package (or per test binary) and shared by every test that only reads it; a test that writes gets a cheap copy of the prepared fixture, not a fresh build.
  Tests that share one fixture must not see each other's writes, so Discussion decides per fixture which tests can share it as is and which need a copy.
  Measured 2026-10-10 on a loaded machine (four runs in flight): one git process costs about 1.3 ms to spawn, a minimal repo (init, config, one commit) about 9 ms, a `cp -a` of a ready repo about 2 ms; `go test -tags integration ./internal/fabricengine/` ran 631 tests in 26.5 s and spawned 50,045 git processes, about 80 per test.
- lyx's own code spawns far fewer git processes for lookups (operator, 2026-10-10).
  In the same `fabricengine` run, by subcommand: `rev-parse` 18,825 (38%), `worktree` 7,299, `commit` and `add` 6,227, `diff` 2,628, `clone` 2,279, `config` 2,092, `push` 1,620, the rest about 9,000.
  Most `rev-parse` calls are lookups (top level, HEAD, git dir, common dir) that can be read from `.git` in-process or cached for the process's life, in `internal/gitrepo`, `internal/gitexec` and `internal/lyxcwd`; that speeds up every `lyx` command, not only the tests.
  The run cannot split fixture calls from calls by the code under test, since both run in the test binary; the measurement goal above splits them first.
- Fixture repositories never auto-gc in the background, and every test waits for the git and lyx children it starts before its cleanup.
- A scan keeps tier 1 and tier 2 free of tmux.
- A static scan enforces `PATTERN-shuttle-stop`: `RemoveStrand` on a shuttle run's strand appears only in `shuttleengine` and the exceptions the entry names (`loomcli`'s removal of the loom driver's own strand, `orchcli`'s removal of the orch strand); the PR-Gate review of `shuttle-send` found the entry false against `orchcli` with nothing to catch it.
