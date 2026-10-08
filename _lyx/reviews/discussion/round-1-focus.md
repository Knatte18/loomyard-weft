---
round: 1
focus:
  - "D7 stop verb: check what it can now skip or let through. It writes `stopped` before removing the strand, and a concurrent `Wait` may overwrite it. Does the Decision bound a `stopped` record over a strand that is still live, and the guid-only path with no run record?"
  - "D7 and the Constraints on `PATTERN-completion-signal`: settle whether attach treating `stopped` as respawn-eligible can bypass the file-contract-first checks, for example a `stopped` run whose output files exist."
  - "D1 signal condition: a `turn_start` after the newest turn end holds the send. Settle what happens when the matching turn end never arrives (crashed session, missing `Stop` hook line). Does the Decision or Open risks state the bound, given that Open risks claim a missing hook line never causes a false busy?"
  - "D6: a re-prompt that fails with `ErrSessionBusy` or `ErrSubmissionNotLanded` spends no gate budget. Settle whether the Decision states what bounds repeated misses other than the run deadline, and whether the boundary-clearing on `turn_start` can loop without progress."
  - "D3 and D4: after a start-time send fails with `ErrSubmissionNotLanded`, the late-arriving prompt may still be consumed before `abandonStartup` tears the strand down. Settle whether the premise that the agent has done no work, so a re-step is safe, holds, and what bounds it."
  - "Consistency across D1, D3, D6 and Acceptance criteria: each Decision's stated behaviour (waits, sentinels, `stopped`, the retired `sendExtraEnters`, the config keys and their zero/negative handling) should be matched by an acceptance criterion and agree with Scope, Constraints and the incident chain in Notes."
  - "Relocation check: look for deliberation narrative in `decision-record.md` (Rationale, Open risks, Auto-mode assumptions) that belongs in `support-log.md`. Before suggesting a move, apply the completeness-before-leanness test, because Plan-Write never reads the support log."
---

Seed focus for round 1, drawn from the attack-surface question, the relocation rule and cross-decision consistency.
Nothing here pre-states a verdict.
