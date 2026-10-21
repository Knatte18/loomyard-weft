# Deploy: drop the shared prod pointer and stop flagging a foreign lyx on PATH

`deploy-prod-cleanup` · Task · bug, deploy · running · Merge-In

## Goal

`update-plugins.sh` (`tools/deploy`) stops pushing a shared `prod` pointer, and its other-lyx-on-PATH check stops asking to remove a foreign binary that happens to be named `lyx`.

## Drop the `prod` pointer

Seen 2026-10-20:

- `update-plugins.sh` builds `lyx` into the machine's Go bin dir, syncs the plugins, then pushes `HEAD` to `origin/prod`.
- `prod` is one pointer on origin, shared by every machine, while each machine's installed binary is local.
- On the Kubuntu machine a deploy moved `prod` to `a1f435dc`; the work laptop still ran a binary built from `c03af73d` while `prod` claimed `a1f435dc`.

Operator decision (2026-10-20): drop the pushed `prod` pointer, since with deploys on two machines it says nothing about what either machine runs.
Only `tools/deploy` writes it and no lyx code reads it.
`CLAUDE.md`'s Production section, which says the script moves `prod` and forbids moving it by hand, follows in the same commit.
The hub-wide orch rules on Bolt are not part of this task's diff; the orch updates them after landing if they name `prod`.

## Foreign `lyx` on PATH

Seen 2026-10-20 on OSL-1046 ([issue #766](https://github.com/Knatte18/loomyard/issues/766)): `update-plugins.sh` printed `WARNING: another lyx on PATH: /usr/bin/lyx - remove it`, and the same for `/bin/lyx`.
`/usr/bin/lyx` is root-owned, from 2024, and is the Ubuntu LyX document processor package; `/bin` is a symlink to `/usr/bin`, so it is one file named twice.
The production `lyx` in `~/go/bin` comes first on PATH, so nothing is wrong.

Expected:

- the check recognises a foreign binary of the same name, for instance by its Go build info or a version probe, and does not ask to remove it;
- it does not report one file twice through a symlinked directory.
