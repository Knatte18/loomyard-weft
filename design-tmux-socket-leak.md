# Tests leave dead tmux sockets in /tmp/tmux-1000

`tmux-socket-leak` · Note · bug, reed

Test runs leave dead tmux socket files in `/tmp/tmux-1000/`; no server runs on them, but the files are never removed.
On 2026-10-05 the Ryzen machine held 659 of them, all but one dead, and new ones still appear on every suite run.

Prefixes seen, most to least common:

- `lyx-warp-bare-LYXHUB-*` (the bulk, over 400)
- `lyx-001-*` (standalone geometry, `internal/standalonegeom/reedgeom.go` `SocketKey`)
- `lyx-contract-*-test-<pid>-<n>` (`internal/reedengine/contract_integration_test.go`)
- `lyx-dotfill-harness-*`, `lyx-attach-harness-<pid>` (`internal/reedcli/smoke_attach_test.go`)
- `lyx-<hash8>` and `lyx-Test*`

`kill-server` stops the server but leaves its socket file; a test that boots a tmux server should remove the socket in its cleanup, ideally through one shared helper, and point `TMUX_TMPDIR` at `t.TempDir()` where it can, so the files never reach `/tmp`.
Also check whether production reed paths (standalone geometry, warp-bare) leave sockets when they are not tests.
