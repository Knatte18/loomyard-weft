```yaml
lens: coverage
outcome: reported
```

Coverage lens: I checked each decision, acceptance criterion and new branch against the card that carries it and the test row that reaches it. I made no extra tool calls; everything below is from the context I already held.

**MEDIUM — D2, Plan-Write's forked gate: no row shows a forked failure spawning a fixer.**
- Card 6 `TestWrapRowGate` has a row where a `SkipForked`-accepted forked failure spawns nothing. It has a row where `Final` runs the inner `Forked` closure.
- No row shows a forked evaluation that fails on something other than the missing lens review, at the boundary, spawning one fixer and re-running `Forked`.
- So the criterion "Plan-Write's forked plan gate is wrapped as its plain one is" has no row that would fail if only `Gate` were wrapped. The evidence is card 6's Intent, which says "its `Forked` when the inner one is set, are wrapped", next to its test row list.

**MEDIUM — D2 `Expired` probe: card 6 compares against a different time than the record names.**
- The record says the probe answers true for a live record "saved before the run's last attach".
- Card 6 compares `SpawnedAt` with `rf.Now` taken when `WrapRowGate` runs, which is at the entry's construction.
- Nothing in the plan ties construction time to the shuttle run's attach. A fixer spawned by an earlier evaluation in the same driver, still live while the writer is mid-turn, reads false.
- `SpawnedAt` is RFC 3339 with whole seconds, so a record spawned in the same second as the build compares ambiguously.
- `TestWrapRowGate`'s row "true for a live record spawned before the wrap was built" pins the near operand, not the record's.

**MEDIUM — card 6: what happens when the fixer cannot spawn or its wait fails is neither specified nor tested.**
- Card 6 names only `VisitDir` and store errors ("returned as the closure's error"), and no row reaches even those.
- It does not say what a failed `Fixer.SpawnOrAttach` does: a transient start failure, or the registration or save stop from card 1. Under shuttle's contract a closure error is an infrastructure fault that ends the gate loop. A transient spawn failure would then end the row instead of re-prompting the writer as today.
- A failed `Fixer.Await` returns with the record still live and the strand never stopped.
- No row covers a spawn error, an `Await` error or a `VisitDir` error.

**MEDIUM — D5: nothing tests the card fixer's prompt, and no body's edit scope or leave-list is checked.**
- Card 12 renders `gatefix-body-card` with the report, the gate call, the `--card` ids and no parent name.
- `TestRunWithFixer` runs over `fakeCardFixer`, and `TestHandsOff` checks only the hand-off decision. No row asserts the gate call, the card ids or the no-parent directive in the card fixer's prompt.
- That leaves two criteria without a row:
  - D4 "the card fixer's prompt carries the no-parent directive".
  - D5 "each caller's rendered fixer prompt names its edit scope, its commit rule, its re-check command and the failures it must leave".
- Card 8 `TestGateFixerSeam` asserts only the re-check commands and the lens review path for the three row bodies.

**MEDIUM — D7 and D4: how the Webster fork decides `CardFixer` is untested.**
- Card 13 sets `CardFixer` in `websterCLI.beginDeps` and `recoverDeps` from the `card` key's `Fixer` together with `pairSlug()`.
- Its rows only render with `true` or `false` passed in.
- The criterion "names the card-gate fixer's reply when the `card` key's `fixer` is set and the worktree has a pair slug, and renders today's text otherwise" has no row that reaches that decision.

**MEDIUM — D1 and D6: how webster's registration is wired is untested.**
- Card 3's `websterCLI.pairSlug()` has three branches: in a hub with an origin, no origin, and standalone. `fixVerifyDeps` then picks between a set and a nil `Register`. No row covers either.
- `TestFixVerifySpawnOrAttach` only injects a `Register` seam.
- So the criterion that ending the pair's processes stops webster's fixer on a pair, and leaves it unregistered on the prime or standalone, is not reached.

**MEDIUM — card 8: a branch with no row, and a function with no test.**
- `gateFixerSeam` refuses "an alias the registry does not hold", and no row in `TestGateFixerSeam` reaches that refusal.
- `pairSlugOf` has three branches: origin found, no origin, and a read error. It has no test. Card 9 and card 10 depend on it too.

**LOW — D7, the Discussion chair is not named in the rows.**
- Card 10's `TestDiscussionTable` rows assert the lead's prompt only.
- The record says the Discussion chair's prompt is included. The chair template wraps the discussion template, but no row renders the chair seat with `fixerOn` true.

**LOW — card 1: some error branches have no row.**
- `TestSpawnOrAttach` covers failures of `FindRun`, `Register` and `Save`, but not a `Starter.Start` failure, the branch that returns "the way forward that the caller's gate re-runs".
- `TestAwait` covers neither an engine `Lookup` error nor an unparsable `SpawnedAt`.

**LOW — card 2: two branches with no row.**
- `TestRenderPrompt` has no row for the friction directive rendering when `NotePath` is set.
- It has no row for a body value named `report_path` being overwritten by the frame's own value.

**LOW — card 6: rows do not assert parts of the stated behavior.**
- The `Final` row with a live record does not assert that the record is marked terminal or that the reply or dead note is folded into a failure.
- No wait-only row covers a dead fixer's note. The record's criterion is "the `clean` findings carry its reply or dead note".
- No row checks that `Reprompt`, `Attempts` and `PassOnCap` are kept on the wrapped entries.
- No row checks that a `Terminal` inner result is returned as it is.
- No row covers `Await` answering running and then terminal (the re-call loop).

**LOW — card 7: a refusal with no row.**
- `env.Setting` is required only when an element carries `fixer`, and no row reaches that nil-`Setting` refusal.

**LOW — card 9: wiring with no row.**
- `loomCLI.specFor` setting `Hooks.PreGoto` has no row.
- Neither does `loomPreGoto` with no location.
- The D8 criterion "a goto past a row with a live fixer leaves no fixer running" rests on the `endGateFixers` unit row alone.

**LOW — cards 11 and 12: two branches with no row.**
- An empty request role, which card 11 says "reads as a role that hands off", has no `TestHandsOff` row.
- `newCardFixer` returning an error, "logged by the runner, which then ends the call as today", has no `TestRunWithFixer` row. Only the turned-down case is covered.

**LOW — D2 deadline bound: argued but not tested.**
- The bound "outruns its deadline by at most one fixer timeout plus three runs of the inner gate" is argued in card 6's Bound and has no row.
- It follows from `Final` spawning nothing, which is tested, so this is noted only.

Further tool calls: none.
