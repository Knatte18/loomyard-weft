```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out, F1 to F6, through the fix-proposal lens.

**LOW: `docs/overview.md`, the api-error sentence**
- Changed text: ``ends the run `api-error` at once;`` followed by `so does a start whose skill-load hold read its cap of API errors.`
- "So does" inherits "at once". The cap ends the start only after the whole back-off schedule. The test fixture sums the 7 back-offs before the 8th error in `cappedHold`, and the `loadHoldMaxAPIErrors` comment puts it at about half an hour.
- A reader takes the capped hold as an immediate end, which contradicts the code.
- A neutral wording fixes it: "a start whose skill-load hold read its cap of API errors also ends `api-error`, after its back-offs".

Proposals F1 to F5 held up against the concrete cases I tried:
- **F1:** `assertCallTimed` compares `StartedAt` with `At` as strings. That is sound, since both are fixed-format RFC 3339 UTC. The `"s"+At` fold row cannot pass vacuously: `"st1"` differs from any later call's start.
- **F2:** the comment option is accurate against `idleGapsOf`, because a fold moves only `LastAt`. The gap hidden inside a fold is now documented, not reported.
- **F3:** the all-outputs-exist row distinguishes the branch under test. Without the early return, the start would end `api-error`, and the row asserts `OutcomeDone` with `NotStarted` false. The gate run was green.
- **F4:** the `"At %d API errors"` check on `notices[0]` is safe. The notice fires once, at the second error, and the capped row has `wantNotices: 1`.
- **F5:** the expected gap map matches the `IdleGap` JSON tags and `idle_gaps` is built from `idleGapsOf`, so a tag rename now fails the test.

I made no tool calls.
