```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency lens over the F1–F3 change set.

No findings.

- The only users of `helperOrphanEnv` are the stand-in's `orphan` condition (line 280) and the two new call sites (lines 1566 and 1586). Both use the new `helperOrphanTier` and `helperOrphanBuild` constants, so no old `=1` value is left anywhere.
- The `helperRoleStandIn` doc comment (line 49) now names both values.
- `_lym/landing/summary.md` matches the tests: two subtests, the 134 and 0 rows, and the shared build.
- The F3 text matches the review's stated protection.
- The `helperOrphanTier`/`helperOrphanBuild` constants sit after the aligned `const` group, with their own comment above them. That is consistent with how the block handles other commented entries.

Tool calls:
1. `grep -rn helperOrphan|GATECLI_TEST_ORPHAN internal/`: to find any other user of the renamed env values, which my context lacked beyond this one file.
