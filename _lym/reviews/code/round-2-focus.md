---
round: 2
focus:
  - "internal/fabricengine/shortnamebinding_clone_integration_test.go, TestCloneHub_ShortnameRecordLifecycle: settle whether every subtest that omits t.Parallel is covered by a comment naming the shared hub fixture it reads, as PATTERN-test-speed requires."
  - "internal/fabricengine/clone.go, CloneHub: settle whether every refusal that runs before the --reset removal (shortname, warp binding, old-order bootstrap guard, non-hub target) leaves the existing hub in place, and whether the --subpath checks that still run after the clone are accurately described in the doc comment and the change description."
  - "internal/fabricengine/*_clone_integration_test.go: settle whether each probe-time refusal under Reset against an existing hub has a test that fails if the removal moves back before the probe."
  - "internal/fabricengine/livestate_*_test.go: settle whether any comment still names resetHub's own pre-flight as the refuser of a non-hub target in the two-argument form, where CloneHub calls checkResetTarget."
---

Focus targets the areas round 1 touched; it adds no cap on severity.
