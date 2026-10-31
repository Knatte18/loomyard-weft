```yaml
lens: coverage
outcome: reported
```

Coverage lens: checking that every decision, acceptance criterion and new branch in the sampler plan has a card and a test row that would fail if it broke.

**Findings**

1. MEDIUM — D1/D6 acceptance criterion "a sampler run whose agent wrote only a board note … passes Describe's gate and lands with no pull request through the existing empty-landing path": no card carries it and no row reaches it.
   - Evidence: no card's Intent mentions an empty diff or a no-commit landing (cards 06, 08, 09 and 10).
   - The criterion is left to existing landingshed and summaryparser tests, which the plan never names.
   - Either name the covering tests (the `ResolvePullRequest` no-commits row, and the description-gate test on a body-only file) or plan a row.

2. MEDIUM — D5/D6 wiring done by `wireLanding` and `buildLoomShed` has no row.
   - Evidence: in card 09, `TestWire_SamplerRecipe` asserts that the four `wireLanding` seams are nil after `wire`.
   - No row asserts that `wireLanding` sets `Sampler.Sessions`, `SessionLive` and `RegisterSession` (the `runproc.RoleSession` registration D5 requires).
   - No row asserts that `buildLoomShed` sets `Sampler.Watch`.

3. MEDIUM — D6 acceptance criterion "the verb's write ends the wait within a second": the real `openRecordWatch` has no row.
   - Evidence: card 04 covers this only with a fake `Watch` channel.
   - Card 09's `openRecordWatch` (watching the record's directory for the base name, nil with a Warn when the watch cannot open) is never driven by any row.
   - So nothing would fail if a `samplerengine.UpdateRecord` write (through `state`) raised no event under the watched name.

4. LOW — Card 04 branches no row reaches:
   - a `RegisterSession` failure is logged and the session kept;
   - a failed record write after a start stops the new strand and returns the error;
   - an unreadable pause record is read as uncovered;
   - a covered wake while a current done flag is present judges nothing (the existing row only checks stop and resume);
   - an error exit (such as a record read failure) stops a still-live session.

5. LOW — Card 10, `TestSamplerDoneVerb`: card 10's Intent says "a failed read of any seam is an error envelope", but only the stamp read failure has a row. The `row`, `currentRow`, record-read and `dirty` read failures have none.

6. LOW — Card 08:
   - `CodePerchGate`'s refusal of an unknown recipe has no row.
   - The error of `codeBudgetZero` when the gate's evaluation fails (for example `sampler.code_review` holding a non-bool) has no row; only the loom.yaml load error is covered.

7. LOW — Card 12: "an absent or unreadable status reads as before, with no error" in `agentActivityFor`. The new rows cover running and blocked states, but no row writes an unreadable run status file.

8. LOW — D1/D3 acceptance criterion "with `code_review` false Code-Bouncer is recorded Skipped, the run goes on to Describe, and no Code-Review row spawns anything".
   - Card 06's `TestSamplerRecipe` asserts only that the gate does or does not hold.
   - No row steps the built shed past Sampler-Session to show the Skipped history entries and the arrival at Describe.

9. LOW (outside this lens) — Card 03's `TestSessionSpec` asserts phrases of the stencil's static prose: `tmux -L`, the sandbox hub, the absence of `git push`. The rubric's "Test economy" flags assertions on a stencil's static prose. D8 and D9 do require the rendered text, so the conflict should be judged deliberately.

No further tool calls made.
