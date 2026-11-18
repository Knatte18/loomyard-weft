```yaml
lens: fix-consistency
outcome: reported
```

Fix-consistency lens: does the change set agree with its siblings, and was any change made in one place without its twin elsewhere?

**MEDIUM: the new overview Shared Decisions bullet contradicts cards 1 and 2**
- Changed text: `00-overview.md`, the new Shared Decisions bullet "A card creates a production function only together with its first caller from a main."
- Summary: stated without qualification, the rule is false for cards 1 and 2. They create functions whose first main caller lands in card 5.
- Evidence:
  - Card 1 creates `ChangedFuncs`. Card 2 creates `Open`, `Reader.TestsReaching`, `Reader.Maps`, `Reader.SpawnMarked`, `Reader.StaleLinkers`, `Fresh` and `ReadPackageMap`.
  - Nothing reaches any of these from a main until card 5's gatecli code imports `internal/testreach`.
  - They are safe only because of the bound in `pattern/PATTERN-test-only-code.md:36-37`: a package no main imports is outside the analysis.
- What would fix it: the bullet should name that exception. The rule binds once a main imports the function's package, so a new package may hold functions until its first importer arrives. Otherwise an implementer or reviewer reading the bullet flags cards 1 and 2.

Outside this lens: none of the F10 ruling's changes is driven by a test. `TestTriageLookup_Lines` scripts `mergeBase`, so the prime's "no parent branch" answer in `printReport`'s wiring has no row. That is a coverage gap, not an inconsistency.

The stale card numbers in `lens-review*.md` are Plan-Write's record of an earlier pass, so I didn't count them as findings.

Tool calls:
- One grep for card-number references in `_lym/plan/lens-review*.md` and `00-overview.md`. My context did not hold the lens-review files, and I needed to check them against the renumbering.
