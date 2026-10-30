# A closed area for lym: bwrap sandbox in stages, off by default

`sandbox-closed-area-build` · Note · feature, reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide

## Seen (operator, 2026-10-30)

Every agent lym starts runs with `--dangerously-skip-permissions`, which many people do not accept, and a repository that states it supports safe sandboxing is worth more than one that does not.
The operator uses lym alone, on Linux (OSL-1046: Ubuntu 24.04; Neve: Kubuntu on Ubuntu 26.04, Plasma 6), and has no interest in other users or in Windows.
The research is `docs/research/sandboxing-options-2026-10-30.md`: it weighs `bwrap`, Landlock, a dedicated Linux user, systemd sandboxing, podman and Claude Code's own `/sandbox`, and the packages and cards each would change.

Two exposures hold today with no area: an agent can reach `/run/user/<uid>/bus` and through it `systemd-run --user`, which starts any process outside lym's tree, and it can type into other hubs' tmux panes through their sockets in `/tmp/tmux-<uid>`, the Models hub's among them.

## Idea

Run the whole lym process tree inside a `bwrap` area launched by lym itself, with the agents free inside it and a short, controlled list of exits.
It is an option the operator turns on, never a requirement: with it off, everything works as it does today, and with it on, nothing starts when the sandbox cannot.
The operator thinks the full design is complicated and error-prone, and doubts it will work without large problems, so it is built in stages with a stop between them.
This is one large note and is split into tasks, or a project, only when the operator decides it goes ahead.

### Before the area: two cheap steps

- Start a hub's tmux server from a Konsole window, not a VS Code terminal: the server inherits the `vscode` AppArmor profile, which grants user namespaces to every agent of the hub.
- A line in `README.md` that every agent lym starts runs with bypass permissions, and that lym belongs on a machine the operator is willing to hand to its agents.

### Stage 1: the filesystem area only

`bwrap` with `--unshare-all --die-with-parent --clearenv` around reed's tmux server and everything under it, started by a new `lym area start` and a supervisor inside.

- The hub is bound read-write at its own path, since the tmux server name, the creel socket name, git's worktree links and Claude Code's transcript directory all derive from absolute paths; `/usr`, `/etc` and `/opt` are read-only, `$HOME` a tmpfs with read-only binds of the Go toolchain, the Claude Code binary and the installed `lym`.
- The area has its own Claude config directory and Go caches bound at the usual paths, with the host's plugin cache read-only on top, so the host's `settings.json` hooks never run in it.
- One host directory holds the tmux socket directory, so `lym reed attach` from outside drives the server inside; tried on OSL-1046 with a tmux server inside `bwrap`.
- `.git/hooks` is read-only; `.git/config` stays writable, since `internal/gitrepo/branchmark.go` writes `branch.<b>.lymLocal` into it.
- Network stays open and there is no broker, so the landing deploy, `lym reed window`, `lym ide` and `lym hibernate request` do not work from inside; the operator does them by hand, from outside.
- A config module `area` with `required`, off by default; `lym area start` refuses on any failure and never falls back to an unsandboxed start.
- Ubuntu 24.04 needs a one-time AppArmor profile for `/usr/bin/bwrap`, loaded by root; the report names the profile, and Ubuntu 25.04 and later ship it, which is not verified on Neve.

This closes the two exposures above and shows early whether tmux, creel and dobby run well inside an area.

### Stop: the operator decides whether stage 1 works well enough to go on.

### Stage 2: egress through a host proxy

No network inside the area; a host-side proxy, reached through a bound socket, allows CONNECT to `api.anthropic.com`, `claude.ai`, `platform.claude.com`, `proxy.golang.org`, `sum.golang.org`, `github.com` and `api.github.com`, with a loopback relay inside the area.
Telemetry and the auto-updater are off by environment.
The GitHub token handed to the area is fine-grained and limited to the hub's repositories; the operator's `gh` login never enters.
WebFetch and the `prowler` skill reach only these hosts.

### Stage 3: a broker for the remaining exits

A host-side process serving a fixed set of verbs and never a command line from the area: the landing deploy (it fetches `origin/main` into its own clone, checks the commit is the tip, builds with the host's Go and installs), `lym reed window` opening Konsole, and `lym hibernate request`.
`lym ide` is refused, since VS Code on the host would run git over repositories the area writes; the host runs no git over the area's repositories at all.

## What is not claimed

Isolation from a kernel exploit, which needs a virtual machine; that data in the area cannot leave through an allowed host; and that a landed change of lym itself cannot run on the host once deployed, which matters on this hub because it develops lym.
Landlock stays a possible inner layer once both machines run a kernel with ABI 9 and 10, and Claude Code deny rules in shuttle's settings are a cheap second line inside the area; the dedicated user, podman, systemd sandboxing and Claude Code's `/sandbox` inside lym runs are left out.
The report estimates the whole at five tasks and eighteen cards.

## Related

Note `creel-cross-hub-mail`: the inbox folder sits outside an area, so reading and writing it goes through the broker.
