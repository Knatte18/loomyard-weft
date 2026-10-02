---
round: 1
ledger:
  - key: bypass-agent-tool-bounds-omit-spawned-agents
    rounds: [1]
    status: resolved
  - key: adopt-bound-false-on-fail-open-registry
    rounds: [1]
    status: resolved
  - key: defer-contract-underspecified
    rounds: [1]
    status: resolved
  - key: soft-hard-trigger-precedence
    rounds: [1]
    status: resolved
  - key: strand-name-criterion-not-marked-live-smoke
    rounds: [1]
    status: resolved
  - key: hub-review-provenance-in-decision-record
    rounds: [1]
    status: resolved
---

Round 1 raised six findings on the orch-live-ready decision record, and the fixer resolved all six in `decision-record.md`.

The four MEDIUM findings concerned unbounded edges and an underspecified veto.
Decisions 1 and 3 now state that subagents and forks inherit orch's permission mode, that an adopted session comes back under it, and that `AllowAgentTool` has no operator switch beyond `permission_mode: prompt`.
Decision 5's Bound now covers only a positively identified live holder, and names what bounds the fail-open registry path: the explicit operator verb, the warning on the log and envelope, and the prime-only transcript check.
Decision 6 now defines `DEFER` as the trimmed final assistant message equal to `DEFER`.
It also rules that a written handoff file wins, that a hard or requested cycle ignores `DEFER`, that the hold counts from the DEFER turn end's read time, and that a reading at or over `threshold_tokens` is always a hard trigger.

The LOW and NIT findings were also fixed.
Decision 7 and its acceptance line now mark the strand-name, parentage and SendMessage check as a live-smoke verification of shipped agent-naming.
The hub-review provenance parentheticals were dropped, and the first auto-mode bullet now credits the hub review.

No finding remains open.
