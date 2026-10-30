# Research: a closed area for lym so bypass permissions is safe

`sandbox-closed-area` · Task · research, shuttle, reed, fabric, batten, dobby, creel, claude, deploy · running · Worktree-Create

## Seen (operator, 2026-10-30)

Every agent lym starts runs with bypass permissions on.
Many people do not accept that, so a repository that says it supports safe sandboxing is worth more than one that does not.
The operator uses lym alone, on Linux (OSL-1046: Ubuntu 24.04; Neve: Kubuntu on Ubuntu 26.04, Plasma 6), and wants no effort spent on colleagues' Windows 11 machines.
The repository may still state that it supports safe sandboxing, and how.

## Idea

Run the whole lym process tree inside a closed area and let the agents act freely inside it, with a short, controlled list of exits.

- Inside the area: the checked-out repositories (the hubs), the Go toolchain and its caches, and lym's own processes (reed and its tmux server, dobby, the orch, every run), running as they do now.
- Outside the area, reachable only through a controlled exit: network to `api.anthropic.com` and GitHub, the GitHub token and the push, the Claude Code config directory and its OAuth credentials, the landing deploy that installs `lym` into `~/go/bin`, the desktop windows and VS Code that `lym reed window` and `lym ide` open, and any git run on the host over a checked-out repository (hooks, `core.fsmonitor`).
- Failing closed: when the sandbox does not start, nothing starts, and there is no fallback to an unsandboxed run.

An outline from a web discussion lists these building blocks: `bwrap` with `--unshare-all`, tmux server inside the sandbox, a cgroup scope per session, an egress allowlist proxy, read-only `.git/hooks` and `.git/config`, a narrow socket API for the verbs that spawn, and an `/sandbox` of Claude Code's own.
Ubuntu 24.04 and later restrict unprivileged user namespaces through AppArmor, so `bwrap` needs a profile there.

## What the research must answer

One darn run, one Opus writer, writes one report to `docs/research/sandboxing-options-2026-10-30.md`.
It reads the lym source itself, and reads current documentation for each method it weighs.

1. Which ways of building the closed area exist for this setup, and what does each cost lym to adopt, in work and in what it breaks?
   The report weighs at most these six methods, each by name: `bwrap`; Landlock through a wrapper; a dedicated Linux user; systemd sandboxing properties on a user scope; a rootless container (podman); Claude Code's own `/sandbox` and its permission and managed-settings features used instead of bypass.
2. For each method: the Ubuntu 24.04 and 26.04 requirements (AppArmor, user namespaces, kernel version), how the lym process tree starts inside it (reed's tmux server and the operator's `lym reed attach`, dobby, creel's daemon and its sockets, shuttle's agent sessions), and what stops working.
3. For each exit on the list above, the smallest controlled way to give it, and which methods can do it.
   In particular the landing `deploy` step, the GitHub token and push, the Claude config directory, and the desktop windows.
4. Which lym code must change for each method, named by package, and an estimate of the work as a count of tasks or cards, never as hours.
5. What a repository can state to say it supports safe sandboxing, and what it can claim only for the method it ships.
6. A recommendation: the method to build first, the order of the rest, and what to leave out.

## Limits

- Exactly one agent run, Opus, effort high; no sub-agent beyond the forks the run's own recipe uses.
- The run may search and fetch as much as the job needs, and may try each sandbox tool (`bwrap`, `landrun`, `systemd-run`, `podman`, `unshare`) with short probes on this machine; no sandbox is adopted, no lym code is changed and no setting of this machine is changed.
- The Models hub and every other hub stay untouched.
