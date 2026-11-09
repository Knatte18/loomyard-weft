```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals F1–F9 carried out as proposed, from my context.

**Findings**

1. **MEDIUM** — F6 text in `internal/dobbyengine/doc.go` ("an anchor move of the prime counts a running refresh as a pass in flight, waits for it, and ends it with the dobby's other processes. What it was building waits for the next `test_map_every` landings") and in `internal/fabriccli/doc.go` ("the move ends it with the dobby; the refresh it was building waits…").
   - The two clauses contradict each other: the move waits until no dobby it will end has a pass in flight, so the refresh has already finished before `EndDobbies` runs.
   - The move record stands from before the wait, so the dobby starts no new refresh meanwhile.
   - Nothing is therefore cut short, and "what it was building waits" is false.
   - The true bounds are that the move waits for the whole refresh, which can be a long build-and-test run, and that it never kills one.
   - The reviewer's proposal assumed an end the wait rules out, and the fix copied it.
   - Evidence: `internal/fabriccli/doc.go` line 164 and `internal/fabriccli/anchordeps.go` `dobbyPassInFlight` (a live child of the dobby that is not push or sync counts as in flight).
   - I did not open the `anchor.go` polling loop to confirm it spans the whole wait.
   - Suggested fix: state only the wait, and drop the "ends it" and "waits for the next landings" clauses in both docs.

2. **LOW** — F1 in `startTestMap`, which logs a Warn on a registration failure and then returns nil.
   - `testMapStep` then zeroes the count and logs "started".
   - Nothing registers the child, so `TestMapRunning` cannot see it, `lym beam stop` cannot end it, and an anchor move does not see it as a registered process.
   - If it outlives `test_map_every` more landings, a second refresh starts alongside it.
   - The doc says "left to end on its own" but names no bound. The reviewer's "within the verb's timeouts" is not in the code I saw.

3. **LOW** — F2 in `tick`.
   - `testMapFailures.Recover` runs only when the step runs, never while an anchor move skips it.
   - An episode that opens before a move therefore stays open through it and logs its recovery after the move ends, which is harmless.
   - The `UpdateState` failure after a successful start returns an error that the episode now swallows into one Warn.
   - With an unregistered child (finding 2) that failure also starts a duplicate on the next tick.
   - The two together are a narrow window.

I found nothing wrong with the proposals for F3, F4, F5, F7, F8 and F9.
- F3: `beamDown` calls `endTestMap` on both the live and the dead dobby path, and `wireCLI` sets it only for the prime.
- F4: the `sh gate` stand-in keeps `$0` as `gate`, and a child that was ended early never writes `done.txt`, so the test would catch the old behaviour.
- F7: the reset-to-zero variant is covered by the new `test_map_every` 0 rows.

**Tool calls:** none. I read nothing further, because my context already held every changed file.
