---
round: 1
focus:
  - "decision-record.md D6 Rationale dates the `llm` name decision 2026-10-06, a day after the dates the record and support log use for the same session: settle whether the date is a factual error or an unexplained forward reference."
  - "decision-record.md D6 and D4: retagging tmux-starting integration files removes them from `cardGateCommand`'s `-tags integration` run, a guard downgraded by this task. Settle whether the Decision that causes it states what now slips past that gate and what bounds it, given the only statement is under Open risks."
  - "decision-record.md D9 production socket removal: settle whether the stated bound (own `-L` key, path is a socket, no server answers) closes the race with a server restarting on the same key, and whether acceptance criteria cover the never-removes-a-live-socket case."
  - "decision-record.md D7: the guard is static and a tier-3 test can reach an LLM through `lyx` without the helper. Settle whether the Decision states what the guard lets through, and whether the `llmkit` `os/exec` exemption in `PATTERN-testkit` is bounded enough to stop widening."
  - "decision-record.md D2 and D1: settle whether the redundancy method is sound as a design, namely coverage subset per top-level test with module-wide `-coverpkg`, the greedy removable set, and the default tag set that excludes `llm` but includes tmux tests run one at a time. Look for a false premise, not missing detail."
  - "decision-record.md D5 and D8: settle whether the five retired PATTERN IDs, the five remaining entries, the Constraints bullets and every ID cited in D5, D7 and D9 agree with each other. Any ID cited as live but retired, or the reverse, is a consistency finding."
  - "Relocation check across the record: look for deliberation narrative in Constraints, Auto-mode assumptions or Open risks that restates a Decision, or belongs in support-log.md. Before proposing a move, confirm whether it carries a requirement the plan writer needs, and extract that first."
---

Seed focus only; the reviewer judges the artifacts itself.
No entry above relies on a call-site enumeration, a missing rejected alternative or a missing plan-writer note, and none asks for exact signatures or `file:line` citations, which the discussion stencil tells the writer not to gather.
