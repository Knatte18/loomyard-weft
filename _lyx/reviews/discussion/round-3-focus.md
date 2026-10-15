---
round: 3
focus:
  - "Attach path and fork audit in decision-record.md D4, D7 and D8: the end-of-run fork audit in internal/shuttleengine/wait.go runs on the run record's ForkSubagents, while the attach Requires check reads the attaching spec. Settle whether D8's bound, that Requires never lets a run skip a check, holds for a record carrying ForkSubagents attached by a spec without it, and what Wait does when that record's engine lacks the fork-audit capability."
  - "Attach refusal in D4 and D8: an attach keeps the recorded engine's live session, and internal/shuttleengine/attach.go removes superseded strands only when no candidate is attachable. Settle whether the refusal's stated way forward can actually clear it, whether the step that ends the strand is named and not denied to agents, and whether a finished run with every output file on disk is refused or harvested."
  - "InterruptSettle in D4: its callers, tenderengine's settle and the orch watcher's settled-key comparison, have no error path. Settle whether the record states the wait used when engine resolution from a guid's record fails."
  - "Webster's default-engine bound in D7 and R4: websterengine audits forks and parses through one engine from its wiring. Settle whether every condition under which that engine can differ from the engine that ran a batch's sessions is stated, with who acts on it."
---

The four entries cover the open keys in the round-2 ledger.
