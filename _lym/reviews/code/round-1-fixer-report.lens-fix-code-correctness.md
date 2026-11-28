```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the code the fix changed.

No findings.

What I walked through:

- **`recordWrite.Call` log switch:**
  - With a stored record, it writes the stored-record log.
  - Record-Seed with an existing record, a missing log and no stored record writes the kept-record log.
  - Record-Write still writes the board log.
  - Record-Seed with a missing record and no stored record returns Stuck before it reaches the switch.
  - `supportLogWithInterview` produces the same bytes as the two literals it replaced.
- **`SeedParentParam`:** the writer (`loomSeedFor`), the reader (`recordSeedDeps`) and the Stuck reason all use `"parent"` as before. The packages compiled and passed in the parent's gate run.
- **New tests against their defects:**
  - The `ReadSeedErrorIsReturned` message assertions fail if the step text or way forward changes.
  - The kept-record log test fails if `boardSupportLog` is written instead.
  - `CancelDuringTheCallWinsOverStuck` fails if the `cancelErr` check before Stuck is removed, because the entry check has already passed when `ReadStored` cancels.
  - `TestRecordSeedDeps_ReadsTheRunsSeed` fails on a wrong param key, recipe, run ID, seed path or slug.
  - The admitted nested-block row fails if nested run-ending blocks are refused.
  - The refused nested-block row fails if the rule applies only to outermost blocks, since nothing else would reject that recipe.

I made these checks by reading the code; I did not run any test with a defect put back in.

Further tool calls: none.
