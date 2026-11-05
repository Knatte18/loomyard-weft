```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fixer's changes agree with the rest of the decision record and support log (fix-consistency lens).

1. **LOW**: Constraints, `PATTERN-shed-recipe-registry` bullet, against the changed D7.
   - Summary: the bullet still says "the mode and model are setting lookups by key", which disagrees with the changed D7.
   - Evidence: D7 now says Raddle-Write reads the key once, records the mode in the run's state, and "every later Raddle-Write session and Raddle-File read that recorded mode, never the key". The bullet should limit the key lookup to Raddle-Write's first spawn.

2. **LOW**: D8, "Raddle-File allows 2 such returns per run and then halts for the parent", against the changed D9 intro.
   - Summary: D9's new "a failing D6 check … counts among its returns" has no twin in D8 or in the D8 criterion.
   - Evidence: in D8, "such returns" still means re-check failures only. The D8 criterion, "a third failure halts the run", covers re-check failures alone.

3. **LOW**: Constraints, the new fact "Codex implements neither `UsageReader` nor `MessagePricer`", against the changed D12.
   - Summary: as written, the fact contradicts D12, which gives the Codex engine a `UsageReader`.
   - Evidence: D12 says "The Codex engine gains a `UsageReader` over the rollout's last `total_token_usage`". The fact needs "today" or "in this tree" next to the claim, so it doesn't read as the end state.

4. **LOW**: D13 heading against the changed D13 body.
   - Summary: the heading "gives each recipe and profile's fixed and per-size cost" no longer matches the body.
   - Evidence: the body now keys each cell by recipe, profile and stored-record flag. It also gives no per-size fit for a run with no size record.

Calls made: none. My context already held everything this lens needed.
