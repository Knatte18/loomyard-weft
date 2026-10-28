```yaml
lens: fix-consistency
outcome: reported
```

Lens fix-consistency.

- LOW — test_integration_test.go, the comment above helperSlotWaitEnv, helperCeilingEnv and helperUnitEnv: it says the helper passes durations to newCommand in place of defaults, but the helper now also passes a fixed 0 as shortageRetryDelay with no comment.
- LOW — test_integration_test.go, the helperRoleStandIn comment: its list of stand-in behaviours omits the new helperShortageEnv branch.

Checked, consistent: verifyFailuresBytes comment, failuresSection doc, newCommand callers, retryOnShortage doc.

Tool calls: 1 grep for verifyFailuresBytes, "quoted whole" and newCommand( across .go and .md files.
