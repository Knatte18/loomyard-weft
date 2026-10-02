---
round: 1
focus:
  - "Decisions 1 and 3 together (bypass plus AllowAgentTool): does each Decision's Bound state what the subagents and forks the orch session spawns now inherit, meaning they run with no permission prompt? And does it say what the operator can still switch off, given that AllowAgentTool has no orch.yaml key?"
  - "Decision 5's registry check fails open (an absent or unreadable registry warns and proceeds): does the Decision state what that lets through, a second live process on the same session id, and what bounds it? Settle whether the stated Bound covers that case."
  - "Decision 6's DEFER contract: is it specified enough to plan from? Check what counts as a turn end that 'carries DEFER' (exact reply or substring), what happens on a soft cycle with both DEFER and a handoff file, and what happens with a handoff file and no DEFER. Also check how the watcher distinguishes a DEFER turn end from an ordinary reply."
  - "Decision 6's trigger precedence: when a reading is at or over both thresholds, which trigger starts the cycle and is recorded? Can a soft-started cycle at a hard-cap reading be deferred, which would contradict 'the hard cap cannot be deferred'?"
  - "The acceptance criterion on strand name, parent and SendMessage after adopt (`<shortname>:orch`): is it backed by a Decision or Scope item, or by behaviour already shipped with agent-naming? Is it stated as a unit or a live-smoke check, so the plan writer knows whether it is new work or verification?"
  - "Leanness: do 'Auto-mode assumptions' and Decision 6's rationale restate Decisions or carry hub-review narrative ('revised by the hub review', 'hub review, 2026-10-02') that belongs in support-log.md? Apply the completeness-before-leanness test before proposing any move."
---

The record is detailed and mostly bounded.
Round 1 should test the four new edges: bypass, the Agent-tool allowance, `--adopt` with its fail-open registry check, and the session's `DEFER` veto.
It should also check that the `DEFER` and trigger-precedence rules are precise enough for the plan writer, who never reads support-log.md.
Call-site enumeration, exact signatures and rejected alternatives in decision-record.md are excluded per the rubric's Do-not-flag list.
