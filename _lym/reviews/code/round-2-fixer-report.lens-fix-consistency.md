```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the F14 change (`orch-role.md` line 156 and the removed summary sentence) agrees with its surroundings.

no findings

- The search of `contracts`, `docs` and `internal` showed no other live text, outside test files, which it excluded, that exempts operator or pull-request feedback from the closed scope.
- The new sentences agree with `orch-role.md` lines 149–155, the role files, the parent directive and the arbiter's later entry.
- `_lym/landing/summary.md`'s "Finalize stays outside the line" bullet still reads correctly without the removed sentence.

Tool calls (1 of 6):
1. `grep` over `contracts`, `docs` and `internal` for feedback and scope-line wording. My context held only a narrower search, which did not cover these phrasings.
