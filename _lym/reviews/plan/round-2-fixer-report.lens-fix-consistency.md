```yaml
lens: fix-consistency
outcome: reported
```

Lens fix-consistency: does the changed text agree with its sibling cards, and was any twin left unchanged? I found one finding.

**[MEDIUM] Card 13 changes how `runBatch` works without editing it, and card 12's description of `runBatch` was not updated**
- **Changed text:** card 13, `resumeStrands` step 4: "A lost attempt, a not-found probe included, retries once on its own rung when the batch has no earlier lost attempt on that rung, else it counts as failed. `runBatch` reads that allowance from the batch's recorded attempts, so a crash never resets it."
- **Evidence:**
  - Card 13's **Edit** list names only `plan:internal/weaverengine#Run`, so `runBatch` is not among the symbols it edits.
  - `runBatch` is created in card 12, which still says "`lost` retries once on the same rung; a second loss counts as a failed attempt". That text counts the loss within one call and never says the allowance comes from the batch's recorded attempts.
  - Card 12's `TestRun` row "a machinery loss ... retries once on the same rung ... and a second loss climbs" has the same in-call reading.
- **Result:** an implementer working from card 12 builds an in-call counter. Card 13 then needs a different `runBatch` without editing it, and the new `TestRun_Resume` row "a crash after a lost attempt retries once on the same rung, and a second loss on that rung climbs" fails against card 12's `runBatch`.
- **Fix:** either say in card 12's `runBatch` bullet that the same-rung retry allowance is read from the batch's recorded attempts on that rung, or add `plan:internal/weaverengine#runBatch` to card 13's **Edit** list.

I checked these twins and they agree:
- Card 5's `Stop` clearing sentence matches the clear sites in cards 10 and 13.
- Card 10's "saves the state through `saveRunState`" is consistent with card 12's rule that every save but the begin's passes `leaseHeld` false.
- Card 18's wiring of `WeaverRun`, its required-seam list and `TestWeaverEntry` agree with card 16 losing its default.
- The `outcome.yaml` keys in card 12 match those card 23 reads.
- Card 23's stuck row still fits.
- `00-overview.md` names none of the changed symbols.

Tool calls:
1. Grepped `_lym/plan/` for every changed symbol (`saveRunState`, `StopRecord`, `resumeStrands`, `outcome.yaml`, `WeaverRun` and others), to find sibling mentions in cards my context had not loaded (03, 14, 17, 26, overview).
2. Grepped card 13's Create/Edit lines and `00-overview.md` for clearing, resume and crash wording, to confirm which symbols card 13 edits and whether the overview restates the changed rules.
