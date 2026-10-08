# Lighter test suite: measured tiers, cheap tmux fixtures, no lingering children

`test-suite-weight` · Task · bug, enhancement, shuttle, reed, fabric

Depends on: [`gate-test-scope`](design-gate-test-scope.md)

## Seen

On 2026-10-08 the integration tier dominated CPU during a Publish verify (`fabricengine.test` alone at 145%), and in the 53 hours before the crash tmux started 9,022 servers and 32,375 panes, nearly all from tier 3 tests, each pane a login shell that sources the operator's profile.
Tests in four packages link the `lyx` binary with cgo on their own, and `TestLoomResumeReconcilesTheHubConfigBeforeArming` flakes on `t.TempDir` cleanup because a git child, most likely a detached auto-gc, still writes into the fixture's object store (#472).

## Goal

- A measurement of every tier per package (time, CPU, peak memory, process count) with `cmd/testtiming`, on a quiet machine, before and after.
- Tier 3 panes run a non-login shell, and tests share one tmux server per package where isolation allows.
- `lyx` is built once per test run and reused across packages.
- Fixture repositories never auto-gc in the background, and every test waits for the git and lyx children it starts before its cleanup.
- A scan keeps tier 1 and tier 2 free of tmux.
