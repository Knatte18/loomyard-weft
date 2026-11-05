# A Codex session cannot reach the creel daemon from its workspace-write sandbox

`codex-sandbox-no-creel-socket` · Note · bug, codex, creel, shuttle

Inside a Codex run on `-a never -s workspace-write`, `lym creel read` and `lym creel post --reply <id>` both fail:

`creel: no daemon answers on /tmp/lym-1000/creel-ly-92073c9b.sock after starting one and 6 connects: dial unix ...: connect: operation not permitted; daemon log .../_bolt/.lym/creel/daemon.log: the daemon log could not be opened: ... read-only file system`

The daemon was up (it had just sent the same strand mail), so the sandbox refuses the AF_UNIX connect.
The Bolt `.lym/creel` directory is read-only there too, so the client's own attempt to start a daemon cannot write its log.
The codexengine doc says the session runs `lym creel read` and `post --reply` from the sandbox and reaches the daemon; it does not.
The reminder `[creel] N unread ... run lym creel read` is typed, but the strand cannot act on it, so agent mail to a Codex strand is unreachable.

Found by the pane/load trial, docs/research/codex-engine-trial-pane-load-2026-11-03.md case 4.