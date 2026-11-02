```yaml
lens: way-forward
outcome: reported
```

Way-forward lens: I checked each refusal, stop and recovery step the cards add, and whether the next step it names can succeed from the state it leaves.

**Findings**

1. MEDIUM — Card 6 (D1, D2): a failed fixer spawn ends the writer's run instead of re-prompting the writer.
   - Card 6 says what happens to a `VisitDir` or store error: it is returned as the closure's error. It says nothing about an error from `Fixer.SpawnOrAttach`. That call fails on a start failure, an unresolved run, a `Register` failure or a `Save` failure (card 1).
   - A closure error ends the evaluation (`internal/shuttleengine/wait.go:1662-1664`). `handleGatedBoundary` then ends `Wait` with `shuttle: gate: %w` (`wait.go:516-518`), and the seat engine passes the error up (`internal/seatengine/engine.go:372`).
   - So a transient start or registration failure ends the row with an infrastructure error and no way forward. D1 says a failure with no fixer reaches the writer as today, and the writer is idle at the boundary.
   - Proposed: on a `SpawnOrAttach` error, return the inner failure as it stands, with a line naming the spawn error, so the writer is re-prompted. Add a `TestWrapRowGate` row for it.

2. MEDIUM — Card 6 (D2 restart path): an `Await` error leaves the record live, and every later evaluation repeats the error.
   - Card 1's `Await` returns errors: an engine lookup error, an events-file read error, a reed status error. Card 6 marks the record terminal only after the re-run, and its `Final` and wait-only closures wait on any live record.
   - So after an `Await` error the record stays live. Every later evaluation calls `Await` again: wait-only entries, `Final` after an attach, and the `Expired`-driven finalize. Each call returns the same error.
   - Nothing marks the record terminal, and no refusal names a way forward. An engine the record names but this lym no longer holds blocks the row for good.
   - Card 12 already handles this for the card fixer: on an `Await` error it stops the strand and marks the record dead `died`. Card 6 has no such branch and no test row for it.
   - Proposed: do the same in card 6 for the fixer entry, `Final` and the wait-only entries, and add a row.

3. MEDIUM — Card 12 (D4, "a cancelled detached run stops its fixer"): nothing watches the lease or signals during the fixer phase.
   - Card 12 step 4 says the fixer phase ends "when the context ends first, by a passed lease or a terminating signal".
   - But the lease watch (`cancelOnPassedLease`) and the signal catch (`catchTerminatingSignals`) are both set up inside `runRequest` and torn down when it returns (`internal/gatecli/detachrun.go:337-346`, `:362-363`).
   - `runWithFixer` waits on the fixer after `runRequest` has returned, so its context (`runDetached`'s `ctx`) never ends on a lapsed lease. A caller that stopped calling leaves the runner and the fixer running for up to `fixer_timeout_min`.
   - The card names no lease watcher for the fixer phase. Its test row cancels the context directly, so it never reaches the lease path.
   - Proposed: `runWithFixer` sets up its own `catchTerminatingSignals` and `cancelOnPassedLease` around the fixer phase. Add a row where a lease written as lapsed stops the fixer.

4. MEDIUM — Card 12 step 1: with no kit, nothing can stop a live fixer before the gate runs.
   - Step 1 says that when it cannot attach ("no kit, or an `Await` error") it stops the strand through `Stop`. `Stop` is a method of the kit's `Fixer` (card 6).
   - With no kit, there is no `Stop` to call. That happens when `cardFixer` turns the request down because the key's `fixer` was since set to `""`, when the slug read fails, or when `newCardFixer` returns an error.
   - The record would be marked dead while the strand keeps editing, and the gate would run over its half-done edits. D4 forbids that.
   - The `TestRunWithFixer` row "an unattachable live record is stopped" cannot be reached through a missing kit.
   - Proposed: build the stopper (shuttle runner, or reed's remove-strand) independently of `handsOff`, and use it whenever a live record exists.

5. LOW — Card 3 (PATTERN-refusal-way-forward): the new `Register` failure in `fix-verify` names no way forward.
   - The new `TestFixVerifySpawnOrAttach` row expects a `Register` failure to stop the strand and return the error. Card 3 does not give the message a way forward, unlike the verb's other spawn errors (`verifyfixer.go:149` and `:155` use "way forward: transient, re-run `lym webster fix-verify`").
   - Card 1's `SpawnOrAttach` returns the joined error, which carries none either.
   - Proposed: the card names the transient re-run way forward, and the row asserts it.

6. LOW — Card 9: no next step is defined when `pairSlugOf` fails during a goto.
   - `loomPreGoto` reads the slug through `pairSlugOf`, which returns a read error (card 8). Card 9 does not say whether that error refuses the goto, and with what way forward, or ends nothing.
   - If it refuses, a transient read of the fabric origin blocks every goto with no named next step.
   - `TestEndGateFixers` covers only the `end` seam.
   - Proposed: refuse with "transient, re-run the goto", or skip ending fixers with a logged warning, stated on the card.

7. LOW — Card 12 step 4: a run can be marked dead while its fixer keeps running.
   - Step 4 stops the fixer and marks the record dead. If the stop fails, the record is still marked dead.
   - The next run of the key then finds no live record and runs the gate beside a fixer that may still be running.
   - Proposed: mark the record dead only after a successful stop, and leave it live otherwise, so step 1's attach-or-stop retries it.

Tool calls I made:
- grep of `internal/shedadapters/multillm.go` and `internal/seatengine/*.go` for the `Wait()` sites, to trace where a gate closure error goes after `wait.go`; my context did not show that path.
- read of `internal/seatengine/engine.go:365-395`, to confirm that a chair's `Wait` error is passed up as an error rather than turned into a re-prompt.
