---
round: 1
focus:
  - "D5 and D6 set aside local commits of an ended entry and tear down its pair: does every Decision that skips, discards or bypasses something state what it can now lose and what bounds it, and does the ended-entry test (D3 ended set plus a local batten run record) have a path that sets aside or removes a live run's work?"
  - "D4 and D10 let a records branch move or push under lock rules (write lock, push lock, try-acquire and defer): does any Decision leave a lock-order or deferral path unbounded, or let a push or move reach a branch a live run, a landing or a held lock is using?"
  - "D10 adds a new creel sender `FABRIC`, a failed-push record and a gone class that refuses a push: does each state what it can now skip or let through, and what bounds it?"
  - "D8's per-call-site behaviour (orch start never refusing, wake's pair pause entries, cleanup refusing on a D3 stop): does a stop at any call site leave a path with no stated way forward, or a guard silently downgraded?"
  - "Relocation check: does `decision-record.md` carry deliberation narrative, such as the Rationale lines or Open risks wording, that adds no requirement or constraint the plan writer needs, and where a requirement sits inside such narrative, is it also stated in Decisions or Constraints?"
  - "Does any Decision ask the run to change this task's own board entry's body or scope, for example D6 step 2's departure from the board entry's local run-record removal, or D11's Preflight change?"
---

Seed focus only; the reviewer still judges everything the rubric names.
Not promoted: missing rejected alternatives and call-site enumeration, both excluded by the rubric's `Do not flag` list.
Not promoted: exact signatures or `file:line` detail, which the writer's spec says not to gather.
