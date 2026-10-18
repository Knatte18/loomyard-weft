---
round: 2
focus:
  - "PATTERN.md and pattern/ against the declared PATTERN changes: does each declared entry's line in PATTERN.md match its declared text word for word, and does the line stay within internal/pattern's entry-length limit?"
  - "internal/fabricengine/pendingflags.go foreignFlagRequest: when a holder respawns a child for a pending code-side flag, does the request carry every path CoalescePushRequest and coalescePushSides require, so the code commit gets pushed?"
  - "internal/fabricengine/spawn.go statusPushRequest and SpawnStatusPush: for a pair where only one side's branch is marked local, can a child be spawned with a request it cannot satisfy?"
  - "internal/fabricengine/add.go final adoption loop: if clearing the second branch's local mark fails after the first succeeded, what state does rollbackAdd leave, and does the comment above the loop match it?"
---

The entries name where to look; each restates the site it depends on.
