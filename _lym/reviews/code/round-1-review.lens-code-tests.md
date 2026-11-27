```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test row that would fail if the behaviour broke.

**Behaviour → test that pins it**

1. **The chair-timeout Stuck reason says the session "timed out and was stopped" and ends with the new way forward.**
   - Pinned by three rows of `TestMultiLLMProducer_MapsTheLeadResult`, `internal/shedadapters/multillm_test.go:360,367,376`.
   - Each row compares the whole reason string exactly.
   - Rewording either part, the "was stopped" clause in `mapOutcome` (`multillm.go:210`) or `timeoutWayForward` (`multillm.go:227`), fails all three rows.
   - Covered.
2. **The `timeoutDetail` wording (elapsed time, limit, last session state) is unchanged.**
   - The same three rows still pin it: limit and state known, all unknown, and the seat's own limit taking precedence.
   - Each row's name still matches what it asserts.
   - Covered.
3. **The new reason claims a timed-out chair has already been stopped.** The diff changes no code here; the claim rests on existing `join` behaviour.
   - Pinned at `internal/seatengine/engine_test.go:35,87`: the timeout row's expectation `chairStopped := ... Outcome == OutcomeTimeout` asserts the stop.
   - Covered.
4. **The new reason claims a resume "starts the row over in a fresh session".** The diff changes no code here; the claim rests on existing shuttle behaviour.
   - Pinned at `internal/shuttleengine/crashresume_test.go:48` ("a run that timed out starts fresh") and `:194` (a timed-out run is not resumable).
   - Covered.
5. **Doc and comment prose in `doc.go` and on the `timeoutWayForward` const.**
   - Static prose, exempt.

**Findings:** none. The diff adds no behaviour without a pinning row, adds no test that duplicates another, and loosens no assertion. The three changed `wantReason` values are exact literals, not computed by the code under test.

Tool calls:
- One grep of `internal/seatengine/engine_test.go` and `internal/shuttleengine/crashresume_test.go` for timeout rows. My context held the production code behind claims 3 and 4, not the tests that pin it.
