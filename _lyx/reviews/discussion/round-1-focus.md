---
round: 1
focus:
  - "D1 and D3: the resume line is built when the session is rebuilt or resumed. Settle whether the decision record says where the role and config come from on that path, so the resume line resolves the same TTL as the launch line."
  - "D4: the whole-set validation refuses a run's start. Settle whether a bad entry can block a resumed or rebuilt session, and whether the Decision states what that refusal bounds, since it is a new start-refusing guard on every role's spawn."
  - "D2 and D4: settle whether a non-string, empty or differently cased value in `claude_prompt_cache_ttl` or a map entry is covered by the stated validation and the acceptance criteria, given the degrading `LoadOrTemplate` loader."
  - "D1 rationale: check the premises the design rests on (reed scrubs `CLAUDE_CODE_*`, the env var outranks the other controls, the version floor) for internal consistency between Decisions, Constraints and Open risks."
  - "Consistency of D5 with Acceptance criteria and `PATTERN-spawn-observability`: settle whether the spawn log requirement is testable as written and whether any Decision introduces a bypass or skip without a stated bound."
---

Seed focus only; every other part of the pair remains in scope for round 1.
