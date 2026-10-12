---
round: 1
focus:
  - "Attack surface in D7: the tackler is called read-only and mutation-free, but is that stated as a bound the system enforces (edit directive, allowed verbs, validation of `goto` and `resume`) or only as an instruction to a cheap agent? Settle what a wrong `resume` or `goto` verdict can make Go do, and what limits it."
  - "Attack surface in D4 and D5: an unwatched Go driver performs its own goto and re-step, and a binary change spawns a fresh `lyx loom drive` and re-steps through batten's bootstrap. Settle that each edge states what it can skip (clean-tree guard, once-per-key caps, the watched marker) and that the bounds survive restart and a stale or spoofed batten-watched marker."
  - "Attack surface in D1 and D10: an absent `child_driver` changes meaning from `go` to resolve-from-`driver_kind`, and `lyx loom resume` gains a spawn branch. Settle whether the LLM-default and the recorded-driver-never-changes guarantees hold at every seeding and resume path the decisions name, and what bounds the new spawn."
  - "Consistency between D3's table and D6, D7, D4, D9 and the acceptance criteria: row order and first-match semantics (the `blocked` rows, the binary-eligible row against the repair-cap and tackler rows, the post-`resume` arbiter row), the three arbiter spawn cases, and the caps counted from settle records. Settle whether any stop class is matched by two rows with different actions, or by none."
  - "D2, D11 and D12 together: the driver lock is held through settle waits, the pause hold defers spawns and re-steps, and a moved run is re-read as a new stop. Settle whether a lock holder can be left behind or a run left unmoved (a crash between lock release and spawn in the D5 hand-over, a pause that outlives a tackler deadline), and whether liveness reads agree across start, resume, batten adoption and the driver-dead notice."
  - "Does `decision-record.md` carry deliberation that belongs in `support-log.md` (hypotheses, history of earlier rounds, comparisons with other board entries), while every requirement and constraint the plan writer needs stays in Decisions or Constraints? Apply the completeness-before-leanness test before any relocation finding."
---

Seed focus only; the reviewer still reads both artifacts in full and may raise anything the rubric allows.
No entry asks for exact signatures, `file:line` citations or call-site enumeration, which the discussion stencil tells the writer not to gather.
