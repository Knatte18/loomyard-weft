---
round: 2
focus:
  - "internal/pauserecord/record.go: does every rewrite of record.json (Update and any helper it calls) go through internal/state, so a key this binary's types do not declare survives, both top-level and on an entry Update leaves untouched?"
  - "internal/pauserecord import set: do internal/pauserecord/leaf_enforcement_test.go, the pauserecord section of pattern/PATTERN-leaf-packages.md and internal/pauserecord/doc.go agree on the admitted imports, and does internal/state import nothing that makes a cycle with internal/lock or internal/fsx?"
  - "internal/pauserecord/hold.go writeMarker: after the atomic-write helper change, is a marker still written whole and atomically, and do the callers in internal/orchengine/pause.go and internal/pauseengine still get the lock and expiry behavior of Update?"
---

Targets the closure of the pause-record rewrite path after the merge.
