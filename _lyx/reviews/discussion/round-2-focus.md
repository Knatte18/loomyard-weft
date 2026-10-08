---
round: 2
focus:
  - "Settle whether `Store.Prune` (`internal/boardengine/store.go`), which removes done entries and rewrites each survivor's `depends_on`, is consistent with the decision record's rule that a running entry is read-only but for its status: does the record list `prune` among the mutating paths, and does one Decision say whether prune passes or is refused when a locked entry depends on a done one, with a matching Acceptance criterion?"
  - "Settle what the lock predicate does with the `abandoned` status, which `Prune`'s doc comment and the board tests treat as terminal, and with the `\"status\":\"active\"` examples in `set-status --help` and `merge --help`: does the Auto-mode assumption about non-run statuses cover them, and is any consequence recorded?"
  - "Settle what happens to the board entry when the claim is written inside Worktree-Create before `Topology.Add` and pair creation then fails: does a Decision name the entry's status afterwards and its disposition, and does Open risks or an Acceptance criterion pin it?"
  - "Settle what a multi-entry write (`upsert-batch`, `merge`) is judged against: does the record define the comparison base, so that a batch listing one slug twice (status null, then a body edit) cannot unlock and edit in one write, and so that Decision 4 and Decision 3's status-only pass agree?"
  - "Settle how a reader refused by the lock checks that no run holds the entry before clearing its status, and whether Open risks or a Decision bound an unlock of a live run."
---

Focus targets the findings that gate or that were fixed by new text no fresh reviewer has seen.
