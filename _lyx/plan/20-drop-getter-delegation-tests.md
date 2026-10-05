# Card 20 — Drop the constant-getter and delegation tests

**Edit:**
- `internal/shuttleengine/claudeengine/session_test.go`
- `internal/shuttleengine/session_test.go`
- `internal/shuttleengine/fakes_test.go`

**Intent:**
Finding F2: three new tests pin no behavior of their own.

- `TestDefaultSkillLoadTimeout` asserts a getter returns a constant; it is deleted.
- `TestRunner_CompactedSince_Delegates` and `TestRunner_LoadSkillAndSkillUnknown_DelegateToCapability` assert calls into lyx's own engine; both are deleted.
  The behavior they stand for is covered at the surfaces that use it: the Claude engine's own `CompactedSince`, `SkillLoadSequence` and `SkillUnknown` tests in `claudeengine`, shuttle's startup skill-load tests, and the orch watcher's reload tests.
  `TestRunner_SessionMethods_ErrorOnPlainEngine` and `TestRunner_LoadSkillAndSkillUnknown_ErrorOnPlainEngine` stay, since a refusal on an engine without the capability is behavior.
- Fake fields and methods the deleted tests alone used (`cyclerEngine`'s recorded asks and scripted compaction time) go; a method an interface still requires stays with a zero answer.

**ImpactSummary:** test-only: three tests and their now-unused fake state removed from shuttle and its Claude engine.

**Commit:** 20: shuttle: drop the constant-getter and delegation tests
