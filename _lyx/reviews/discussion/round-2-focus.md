---
round: 2
focus:
  - "In `_lyx/discussion/decision-record.md`, check the Decision on the per-role TTL map's merge semantics against the loader: `shuttleengine.LoadConfig` calls `configengine.LoadOrTemplate`, and `yamlengine.FillMissing` appends template keys a present mapping lacks unless the key is a declared open map (`configreg.Module.OpenMaps`, `batcher.ConfigOpenMaps` as precedent). Settle whether the Decision, Scope and Acceptance criteria are consistent with that behavior."
  - "In `_lyx/discussion/decision-record.md`, check the Decision that refuses a bad TTL value on spawn: it checks the whole configured set in `claudeengine.Prepare`, so one bad entry blocks every role's new run. Settle whether it states what the guard can block, what it cannot touch (an already started or rebuilt session) and what bounds it."
  - "Check that the Decision on where the resolved TTL is logged names the package that logs it, given that `shuttleengine` does not interpret the TTL and the resolved value exists only inside `claudeengine.Prepare`, and that the acceptance criterion gives a test an observable."
  - "Check that the Goal and the launch and resume line Decisions agree on when the TTL is resolved, given that `claudeengine.Prepare` builds both lines once and reed replays the stored resume line."
---

Focus entries name where to look and which question to settle; they cap no severity.
