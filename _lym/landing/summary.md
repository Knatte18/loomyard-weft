# Research: a closed area for lym so bypass permissions is safe

Adds `docs/research/sandboxing-options-2026-10-30.md`, a report on running lym's whole process tree inside a sandbox on Linux so that agents keep `--dangerously-skip-permissions` inside it and reach the rest of the machine only through a short list of exits.
No code, config or other doc changes.

## What the report finds

- **Recommendation:** build the closed area on `bwrap`, launched and supervised by a new `lym area` module, with reed's tmux server as the area's console and a host-side lym process as its only door: an egress proxy with a host allowlist and a broker serving a fixed set of verbs (deploy, desktop window, hibernate).
  Estimated at five board tasks and about eighteen cards.
- **The operator's attach keeps working:** a probe on OSL-1046 ran a tmux server inside `bwrap --unshare-all` with `/tmp` and `/run/user` as tmpfs and drove it from a tmux client outside; inside, the session bus, other hubs' tmux sockets and the network were unreachable.
- **Ubuntu 24.04 needs one root step:** an AppArmor profile for `/usr/bin/bwrap` (AppArmor's `bwrap-userns-restrict`); 25.04 and later ship it.
  Unconfined probes on OSL-1046 show `bwrap` and `unshare` refused without it.
- **systemd user-scope sandboxing fails open on Ubuntu:** the probe started a user service with `ProtectHome=read-only` and `PrivateTmp=yes` and applied neither, and `IPAddressDeny=any` was ignored.
- **Landlock does not close the area on kernel 7.0:** pathname Unix sockets (the session bus, other hubs' tmux servers) and UDP stay reachable until Landlock ABI 9 and 10.
- **Claude Code's `/sandbox` is not a closed area for lym:** lym's own processes and hooks stay outside it, and the loom driver's Bash must reach the hub's tmux socket, which is itself an escape; it also cannot nest inside a `bwrap` area.
- **Two exits stay open by design:** landed and deployed changes to lym itself run on the host, and data can leave through an allowed host such as `github.com` with an attacker's token.
- **Today's exposure, with no area:** agents can reach the session bus and other hubs' tmux sockets, and a hub booted from a VS Code terminal inherits the `vscode` AppArmor profile, which lets every agent create user namespaces.

## Design decisions

- The report weighs exactly the six methods the board entry names; Anthropic's `sandbox-runtime` (`srt`) and a systemd system unit appear only as variants inside the `bwrap` and systemd sections.
- Probe results state the AppArmor label they ran under, because this session's tree carries the `vscode` label and a fresh login shell does not; `landrun` and `podman` are not installed here and their sections rest on documentation alone.
- Exits are judged against a threat model stated first in the report: a hostile agent session must not act outside the area, read secrets outside it, or persist code that later runs outside it.
- Host git over a repository the area writes is judged to have no safe grant, since lym needs `.git/config` writable (`internal/gitrepo/branchmark.go`, worktree tracking); the deploy broker therefore builds from its own host-side clone of `origin/main`.
