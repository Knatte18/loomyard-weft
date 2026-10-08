---
round: 4
focus:
  - "Stub providers standing in for claude in internal/loomcli (integration_driverbootstrap_test.go integrationWriteStubDriverScript, smoke_driverstrand_test.go writeStubDriverScript): the send path in internal/shuttleengine/send.go (awaitIdleSession, reached through sendVerified from loadSkillTurn in run.go) requires the pane to be both StartupReady and claudeengine.IdleSession. Settle whether each stub's output satisfies both before every verified send, including the pointer send after the skill-load turn, and whether any other stub or fake provider in the tree prints only claudeengine.ReadyFooterFixture."
  - "Any new idle-box fixture added in internal/shuttleengine/claudeengine: settle whether it is pinned by a row in the existing IdleSession and Startup table tests rather than a separate test, per PATTERN-test-economy."
---

Focus targets the round 3 fix surface only; the remaining diff was reviewed without finding in round 3.
