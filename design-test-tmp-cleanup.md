# lyx cleans up its own temp files

`test-tmp-cleanup` · Task · bug, fabric, loom

## Problem (operator, 2026-10-11)

lyx must clean up its own temporary files while it runs; it may not leave garbage behind.
On this machine `/tmp` is a RAM tmpfs, and the leftovers had grown to 8.5 GB, enough to make hibernation fail (kernel `Error -12 creating image`, `Cannot allocate memory`) three times on 2026-10-11 and once the evening before.

## Seen (2026-10-11)

Leftovers in `/tmp`, all older than two hours:

- 220 `loomcli-smoke-lyx-*` directories of about 34 MB each (a built `lyx` binary), about 7.5 GB in all
- over 20 000 `gitkit-gitconfig-*` files
- over 2 000 `hubforge-bare-*` directories
- 171 `gitkit-repo-*` directories

The orch deleted every one older than two hours by hand; `/tmp` went from 8.5 GB to 1.1 GB.

Sources found:

- `internal/gitkit/hermetic.go`: the hermetic git config file from `os.CreateTemp("", "gitkit-gitconfig-*")`, once per test binary, never removed.
- `internal/gitkit/gitkit.go` `buildRepoTemplate` and `internal/hubforge/hub.go` `buildBareTemplate`: once-per-binary fixture templates from `os.MkdirTemp`, never removed.
- `loomcli-smoke-lyx-*`: a `lyx` binary built for the `internal/loomcli` smoke tests; the creating site was not found by a literal grep.
- Other `os.MkdirTemp("", ...)` sites to check: `internal/testkit/lyxbin`, `internal/gatecli/prebuild.go` and `cmd/testtiming` (`lyx-prebuilt-`, `lyxbin-`, `ttr-`, `ttt-`, `testtiming-redundancy-`).

## Wanted

- Every temp file or directory lyx or its tests create is removed when the creating process ends, including on test failure and panic; once-per-binary fixtures are removed in `TestMain` after `m.Run`.
- Prefer a per-run root under one lyx-owned directory, so a killed process's leftovers can be swept by age at the next start.
- A scan or test that fails a new bare `os.MkdirTemp("", ...)` or `os.CreateTemp("", ...)` without a matching cleanup.
