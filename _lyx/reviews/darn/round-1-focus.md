---
round: 1
focus:
  - "internal/reedengine/probe.go: does the private probe socket close the race the board entry names, or can a concurrent boot still reach a transient server on the hub socket (every code path that runs `list-commands`, including resume)?"
  - "internal/reedengine/probe.go: cleanup of the probe key's socket file and the error path when `removeSocketFileOnceGone` fails or the probe itself fails; does the Windows branch behave as before?"
  - "Socket path length: does the claim that the probe key's path fits wherever the hub's passes reed's path-length check hold against the code that forms hub keys?"
  - "Claims in _lyx/landing/summary.md (key length, 2s socketGoneWait, no change to Windows, cost of one extra tmux server per up/resume) against the diff."
  - "Docs and tests: internal/reedengine/doc.go matches the new probe behavior; the new tmux-tier test covers the behavior without duplicating an existing test, and its non-parallel claim holds."
---

Seed focus from the board entry (flaky `TwoWorktreesBootedConcurrently`) and the change description; the diff itself was not read in this pass.
