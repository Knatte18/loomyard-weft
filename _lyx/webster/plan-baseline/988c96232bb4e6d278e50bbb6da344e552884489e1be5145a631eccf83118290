# Card 20 — Drop the constant-getter and delegation tests

**Edit:**
- `internal/shuttleengine/claudeengine/session_test.go`
- `internal/shuttleengine/session_test.go`

**Intent:**
Finding F2: new tests pin no behavior of their own: a constant getter and the Runner's pass-throughs.

- `TestDefaultSkillLoadTimeout` asserts a getter returns a constant; it is deleted.
- `TestRunner_CompactedSince_Delegates` asserts a pass-through to lyx's own engine; it is deleted.
  The engine's own `CompactedSince` tests in `claudeengine` cover the detection, called on the engine directly, and `TestRunner_SessionMethods_ErrorOnPlainEngine` covers the Runner's refusal; the Runner's pass-through keeps no test of its own, by choice.
- `TestRunner_ContextTokens_Delegates` asserts the same kind of pass-through and is deleted too: no Runner pass-through keeps a test of its own, and the `SessionIdle`, `ClearSession` and `CompactSession` tests already cover the success path through the engine's capability.
- `TestRunner_LoadSkillAndSkillUnknown_DelegateToCapability` is replaced, not just deleted: the startup and watcher tests never call `Runner.LoadSkill` or `Runner.SkillUnknown`, so nothing else covers their strand guards or the typing of the load sequence into the pane.
  The replacement is one test of the Runner's own behavior: for an unknown guid and for a dead strand, both methods refuse and type nothing; for a live strand, `LoadSkill` types the engine's load sequence, and `SkillUnknown` answers true for the skill the pane capture names and false for another.
  It reuses `skillFakeEngine`, which stays.
- `TestRunner_SessionMethods_ErrorOnPlainEngine` and `TestRunner_LoadSkillAndSkillUnknown_ErrorOnPlainEngine` stay, since a refusal on an engine without the capability is behavior.
- Fake fields and methods the deleted tests alone used (`cyclerEngine`'s recorded asks, scripted compaction time and scripted token reading) go; a method an interface still requires stays with a zero answer.

**ImpactSummary:** test-only: `TestDefaultSkillLoadTimeout` removed from `claudeengine`'s `session_test.go`, and in shuttle's `session_test.go` two pass-through tests removed, one delegation test replaced by a guard-behavior test and the now-unused `cyclerEngine` fake state (`compactedAt`, `sinceAsks`, `tokens`, `known`) removed.

**Commit:** 20: shuttle: drop the constant-getter and delegation tests
