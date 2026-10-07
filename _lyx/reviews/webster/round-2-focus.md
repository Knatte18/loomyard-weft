---
round: 2
focus:
  - "Comments the diff adds or changes in internal/fabricengine, internal/fabriccli, internal/statuscommit, internal/lock and internal/loomcli: does every sentence start its own line, with a further break only at an independent-clause boundary, per PATTERN-comment-line-breaks?"
  - "internal/statuscommit Deps.Push comment and its push-warns bullet: does any claim hold for both the loom and batten Push wirings, given that batten's PushAnchored returns a bare rejection and a lock expiry names no side?"
  - "internal/lock AcquireWriteLockWithin: is every branch reachable given what the flock TryLockContext call can return?"
  - "The new push helpers in internal/fabricengine (PushPairAnchored and its retry helper): is each behavior the diff adds, including the unborn-HEAD skip and a second push rejected again, reached by a test?"
  - "The lyx loom help text for commit-records: does it match the verb's current observable behavior as documented in docs/overview.md?"
---

Entries name where to look; severity and verdict stay with the reviewer.
