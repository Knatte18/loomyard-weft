---
round: 1
focus:
  - "Cards 1-5: does every resolver build its ModelChoice through `shuttleengine.NewModelChoice`, with timeouts and spawn budgets kept beside the choice and never inside it, and no composite literal bypassing the constructor?"
  - "Cards 7-9: does webster record each session's engine in `state.json`, read every session through that engine, and refuse in `run` and `recover-batch` before any mutation happens?"
  - "Cards 11-13: does a Start requirements refusal name the caller's model key, does each new refusal land its `contracts/specs/refusal-spec.md` row, and do Master and the loom driver declare the requirements the plan names?"
  - "Card 14: does the PATTERN-model-choice-mapping scan catch the bypass it describes, and is the invariant written into PATTERN.md in the same commit?"
  - "Test economy across all cards: does any new test duplicate an existing test, and does any added behavior lack coverage?"
---

Focus is where to look first; the full range and rubric still apply.
