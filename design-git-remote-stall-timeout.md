# Git network calls get a stall bound and one retry

`git-remote-stall-timeout` · Task · bug, fabric, batten · running · Code-Bouncer

## Seen (orch, 2026-10-23, OSL-1046)

Two batten runs started at 18:54 each hung on a git call to GitHub: `orch-handoff-timeout-delivery`'s Worktree-Create on `git ls-remote --heads origin refs/heads/<slug>-weft`, and `adopt-owner-write-stale-manifest`'s `lyx fabric ... push`.
Each `git-remote-https` held one TCP connection to 140.82.121.4:443 with 395 bytes unacknowledged in its send queue, the size of a TLS opening, and no answer for over two minutes, while a fresh `git ls-remote` from the prime to the same repository answered at once.
TCP retransmission would have held each call for about a quarter of an hour; nothing in lyx bounded it.
Ending the two `git-remote-https` processes by hand failed the calls, and batten retried and went on.

## Idea

Every git network call lyx makes carries a stall bound, such as git's `http.lowSpeedLimit` and `http.lowSpeedTime`, or a timeout on the command, and is retried once on a stall, so a dead connection costs seconds instead of minutes.
