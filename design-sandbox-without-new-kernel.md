# Study: close the bus and tmux exposures without a kernel newer than 7.0

`sandbox-without-new-kernel` · Task · research, shuttle, reed, claude · running · Describe

## Aim

`docs/research/platform/landlock-area-2026-11-23.md` found that on Landlock ABI 8 (kernel 7.0, Neve) an agent still reaches the session bus, and through `systemd-run --user` escapes the area, and reaches other hubs' tmux sockets.
Pathname-socket control arrives with ABI 9 in Linux 7.1.
Find what else closes those two exposures now, on Neve's 7.0 and on OSL-1046 (Ubuntu 24.04, no AppArmor profile for user namespaces, sudo tightly limited), with no root at run time.

## Routes to weigh

- seccomp user notification on `connect()`, the route the Landlock report named but did not probe: a supervisor in lym that makes allowed connects itself (`SECCOMP_IOCTL_NOTIF_ADDFD`), refusing `io_uring_setup` and the 32-bit ABIs.
- Taking the exposures away instead of guarding them: the session bus left out of the agents' environment and its socket made unreachable for them; each hub's tmux server on a socket in a directory other hubs' agents cannot reach; whether same-uid processes can be kept apart that way at all.
- A separate Unix user for agents, if it needs root only once at setup.
- Landlock ABI 8 combined with any of the above, and PreToolUse hooks as a second line.
- Anything else found that needs no kernel upgrade, no user namespaces and no root at run time.

For each route: whether it closes each exposure, what it costs lym to own, what breaks, and whether it works on OSL-1046.

## Bounds

At most two probe programs, each run on Neve at most three times; routes not probed are judged from documentation and marked unmeasured.
No change to lym's code.
The report goes to `docs/research/platform/sandbox-without-new-kernel-2026-11-23.md`, ending with a ranked recommendation and the tasks it would take.
