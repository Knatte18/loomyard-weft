```yaml
lens: fix-code-correctness
outcome: reported
```

Task: correctness check of the code the fix changed (fix-code-correctness lens).

**LOW**: `internal/beamcli/dobbydeps.go`, `tellRefusal` (post moved outside the lock). Under concurrent stamps of one refused run, the refusal mail is no longer guaranteed to arrive before the first stamped mail.
- **Case:** caller A sets `told[slug]`, unlocks and blocks in `s.post`. Caller B for the same slug sees `told[slug] != nil`, returns at once, and can post its `; autonomy: false` mail first. If A's post then fails, B's mail has gone out with no refusal mail before it, and only a later refused read retries.
- **Claims affected:** the `stamp` doc says "telling the orch of a refused value first". The test comment says "before the first mail it stamps". `_lym/landing/summary.md` says "the first stamped mail is preceded by one mail to the dobby's orch". The old code, which held the lock through the post, kept that ordering.
- **Weight:** this matters only if the dobby's gate-hold, run-notice and watch posts can run on different goroutines. I did not check that. The wording could be narrowed instead of changing the code.

No other findings:
- **`tellRefusal` paths:**
  - A successful read ends the episode.
  - A failure with no refusal leaves it as it is.
  - A failed post drops only its own episode, because `&line` is distinct per call.
  - Reading from the nil map is safe.
- **`AutonomyWithRefusal`:** all four return paths give the right `(level, refusal, read)`. Its callers are `Autonomy`, `stamp` and the hubgeom tests, and all are updated.
- **New tests:**
  - The refused-again step in the notice test fails if a successful read does not end the episode.
  - The `unattended` row fails if `runAutonomy` compares against `AutonomyOn`.
  - The new watch lines do not disturb the `watchRecords` count.

Note, not a finding: no test fails if `tellRefusal` goes back to ending the episode on any nil refusal (the F5 defect). The hubgeom assertions pin only the `read` flag, not how `tellRefusal` uses it.

Tool calls: none.
