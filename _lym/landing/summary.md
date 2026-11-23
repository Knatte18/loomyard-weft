# Study: a lym-started Landlock area instead of bwrap

Adds `docs/research/platform/landlock-area-2026-11-23.md`, which answers whether lym can close the two exposures in note `sandbox-closed-area-build` by applying Landlock to its own process tree, with no `bwrap`, root or AppArmor profile.
It also adds the probe the report measures with, `docs/research/platform/landlock-probe/main.go`, and a line in `docs/research/README.md`.
No lym code changes.

## Findings

- Neve's kernel 7.0.0-31 has Landlock ABI 8 (measured); OSL-1046's 7.0.0-42 is the same series and ABI 8 by its kernel version, unmeasured.
  The kernel refuses both candidate pathname-socket bits with `EINVAL`.
- Under every right ABI 8 offers, a restricted child still connected to `/run/user/<uid>/bus` and another hub's tmux socket, listed that hub's sessions and ran `systemd-run --user`.
  The last escapes the domain and so undoes every filesystem rule: on ABI 8 the area is a guardrail, not a barrier.
- What Landlock does deliver, measured: `TSYNC` restricts every thread of a Go process; children inherit across `exec`;
  reads and writes outside the allow list fail, SSH keys, the `gh` token and lym's credentials file among them;
  signals and abstract sockets outside the domain are refused; `sudo` fails under `no_new_privs`; a nested ruleset cannot widen the first.
- git, a Go build, `lym board list`, `claude --version` and the hub's own tmux worked under an enumerated allow list, which the report tabulates path by path with why.

## Design decisions

- The probe is one Go file with `//go:build ignore`, so no package build, vet or test run sees it; it runs with `CGO_ENABLED=0 go run`.
  It connects without sending a byte and removes every file it creates; it ran three times on Neve, the bound the task sets.
- Per the two parent decisions added after Discussion, the allow list lives in one file per hub, `~/.config/lym/area-<hub key>.yaml`, outside the hub and every writable path.
  The report shows the probe failing to create it from inside, and states that a changed list takes effect only at a start from a host shell.
  It names the `PATTERN-config-layers` change such a file outside the hub needs.
- The recommendation: build no Landlock area on ABI 8; add PreToolUse rows in `internal/shellguard` for the bus commands and other hubs' tmux servers now;
  re-run the probe on each new kernel; leave seccomp user notification, the one no-root barrier on 7.0, unless the operator wants to own its supervisor.
  The report lists the five tasks the area takes once the kernel controls pathname sockets.

## Merge danger

**Door:** two-way
**Blast radius:** docs
