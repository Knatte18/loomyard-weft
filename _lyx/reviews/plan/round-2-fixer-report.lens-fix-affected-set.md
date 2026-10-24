```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Each finding's set was rechecked; none was left unchecked:
- **F1 (the callers of `filterArgument` and `renderTierCommand`):** `filterArgument` is called only from `renderTierCommand`. `renderTierCommand`'s callers are `runInvocations` (`internal/gatecli/tierrun.go:136`) and `TestRenderTierCommand` (`internal/gatecli/tiers_test.go:429`). Card 8 lists both under Edit, so the set is covered.
- **F2 (seed test classes stated without test attributes):** `CircleTests` and `OtherTests` are the only bare "class" seed classes, both in `09-dotnet-scoper.md:58`, and the fix states their attributes. Card 13's `CircleTests` and `SquareTests` are already called "test classes". Card 12's `Unused.cs` row reuses card 9's tree. The set is covered.
- **F3 (references to renumbered cards):** every card-number mention outside the headings now points at the right card:
  - `02:37` → 12 (shared build)
  - `06:11` → 11 (inert)
  - `09:46` → 12
  - `11:26` and `11:34` → 10 (carry and fingerprint)
  - `11:30` → 7 (derivation)
  - `12:42` and `12:73` → 9 (dotnet scoper)
  - `12:43` and `12:46` → 10

  No stale mention is left. The `lens-review.lens-*.md` files were left out on purpose, as historical reports.

Tool calls:
1. A grep of card-number mentions across the plan cards, to rerun F3's search, which my context held only from before the renumbering edits.
2. A grep of C# test class wording across the plan cards, to rerun F2's search for seed classes beyond the two the review named.
3. A grep of `filterArgument` and `renderTierCommand` callers in the plan and in `internal/gatecli`, to rerun F1's caller search, which my context did not hold.
