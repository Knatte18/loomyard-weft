---
round: 1
focus:
  - "internal/hibernateengine/carrier.go, watch.go, verbs.go: does the carrier run only in the prime's dobby, and does no beam's dobby carry or vote on a request? Check how the prime is identified."
  - "internal/hibernateengine/request.go, carrier.go: is the request kept on disk and taken up again when the prime's dobby starts after a death or a late start? Check what happens to a half-handled or already-fired request on restart."
  - "internal/hibernateengine/watch.go: does `--after-all` wait on the runs of every beam through `battenrecipe.PastLand`, and does the every-session-paused wait, fork rule included, move over unchanged?"
  - "internal/hibernatecli/spawn.go, cli.go, deps.go, reconcile.go: does `lym hibernate request` no longer start a daemon of its own, and is nothing left behind (spawn paths, reconcile entries, dirs, cancel and status paths) that still assumes one?"
  - "internal/beamcli/dobbydeps.go, dobbywatch.go, internal/fabriccli/anchor.go, anchordeps.go: does a hub pause leave the dobby that waits on the request outside it, so the pause the request adds never stops the wait? Check the pause scope and the wake path."
  - "internal/cleanupcli/reconcile.go, internal/lymdirs/dirs.go, pattern/PATTERN-daemon-wakeups.md, docs/overview.md, doc.go files: do the doc and pattern edits match what the code now does, and does the repository's constraints document name other docs this change should have updated?"
  - "internal/hibernateengine tests and internal/beamcli tests: is each new behavior covered once, with no duplicate of an existing test, and is any behavior the diff adds left without coverage?"
---

The decision record holds only the board entry under `## Goal`; its other sections are empty, so the entry's four decisions are the measuring stick.
