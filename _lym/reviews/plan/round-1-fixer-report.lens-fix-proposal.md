```yaml
lens: fix-proposal
outcome: reported
```

I checked each proposal the fixer carried out as written, looking for a concrete case where the proposal itself is wrong.

**MEDIUM: card 8, TestRunAdopt (F1 as proposed).** The rows "a record that ends `stopped` exits non-zero with its way forward" and "one that reads `interrupted` exits non-zero naming the re-run and `lym adopt --discard`" were cut on the ground that TestFollow already pins them. It does not. `follow` returns `(adoptrecord.Record, error)` and never an exit code. Card 8's runAdopt step 3 has its own branch for this: `output.ErrFields` and a non-zero exit for `stopped`, an error for `interrupted`. The only kept mapping row is `done` → 0.
- Concrete case: a `runAdopt` that prints the `stopped` record through `output.Ok` and exits 0 passes every planned row.
- Evidence: `_lym/plan/lens-review.lens-row-validity.md:30-32` says this mapping (exit 0 for done, non-zero for stopped or interrupted) needs a verb-level row, because "TestFollow returns a record or an error and never an exit code". The fix undoes that earlier resolution.

**MEDIUM: card 8, TestRunAdopt's last row (F1 as proposed, the cap half cut).** What is left is "an empty strand follows uncapped without calling `KeepWarm`". No row now pins that runAdopt passes `KeepWarm`'s cap to `follow` for an agent strand, or that `still running` leads to a non-zero exit.
- TestFollow's cap row feeds the cap in directly, so it never checks the `followCap(KeepWarm, strand)` wiring in `runAdopt`.
- Concrete case: a `runAdopt` that always passes cap 0 makes every agent-session caller wait with no bound, and every planned row still passes.

**MEDIUM: card 8, the new follow bullet for a nil `exited` (F3 as proposed).** The proposal ends the follow only when the record names another pid. An absent record under a nil `exited` still loops forever, because the "reads once more, then errors" branch fires only "once `exited` is closed", and a nil `exited` never closes.
- Concrete case: a plain-shell caller follows an existing worker (the follow verdict). That worker dies, leaving its record `running` with a dead pid. Before the follower's next wake (up to `followWakeCeiling`, 8 s), another shell runs `lym adopt --discard`. Holding the free lock, `--discard` removes the unfinished record. The follower then reads no record at every wake, with no cap and no closing `exited`, so it never returns.
- Remedy: the nil-`exited` rule should also end the follow on an absent record, with the same error as the exited-and-no-record case.

**LOW: card 6, the outcome-part definition (F2 as proposed).** "only the `slugOutcome` fields that step set" defines what a part contains. It still does not say how a step tells which fields it set, and that was the review's premise.
- With `omitempty` JSON tags, comparing the outcome before and after a step cannot record a field the step set to its zero value or to the value it already held.
- Concrete case: a step clears an earlier part's per-slug `error` to "". The per-field merge keeps the earlier non-empty value.
- Remedy: say that each step writes into its own fresh outcome and per-slug outcome, and that this outcome is the part.

Checked with no finding: F4, F5, F6, F7 (card 7 half), F8, F9 and F11. For F11, `fakeStarter.StartSession` alone appends "start", and the dobby comes up through other seams, so "no `start` among the recorded calls" holds under `SkipOrch`.

Tool calls made:
- Read `internal/orchcli/start_test.go:40-60,176-200` and grepped the `OwnedElsewhereError` / `ownedElsewhereFor` sites, to check whether "start" in `h.calls` comes only from the orch session starter (F11) and where the battencli refusal fires (F6). My context had only grep lines for these, not the recorder code.
