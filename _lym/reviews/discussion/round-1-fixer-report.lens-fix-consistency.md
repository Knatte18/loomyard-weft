```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the decision-record fix set agrees with the sections around it (fix-consistency lens).

**MEDIUM — D3, new filed-run refusal sentence.**
The refusal covers `add` and `replace` only.
But D3's unchanged sentence "`lym loom decision store` is admitted on the same runs" still admits `store` on a raddle run whose filing is done.
`store` copies the committed run record through `StoreDecisionRecord` onto the run's entry, and that entry is now the family project.
So `store` can still overwrite the record Raddle-File wrote, which is the F5 premise.
Evidence: D3 "`lym loom decision store` is admitted on the same runs, with its caller check unchanged"; `decisionStoreVerb` in `internal/loomcli/decision.go` calls `perchFilingRefusal` nowhere.

**MEDIUM — D2, new seed-trailer sentences, against Open risk 3.**
Open risk 3 says a failed board copy followed by a re-entry into the run's Motif block makes the seed rewrite the run's record from the stale board copy.
With the seed commit now carrying the trailer, the history is cut at that stale seed.
The dropped-entry check then reports no `added-decision-dropped` for the entries the stale copy lost.
The open risk does not mention that the check no longer catches the overwrite.
Evidence: D2 "`seedDiscussedRecord`'s commit carries the same trailer … So a run seeded onto an adopted weft branch reads no version from before the seed"; Open risks, third bullet.

**LOW — D2 heading, D2 Bound, and D6 spec sentence.**
These still say the cut is at "the newest replace" only:
- the heading "The history check reads versions back to the newest replace only";
- D2 Bound "an entry recorded before the newest replace and absent from it is no longer a finding";
- D6 "the dropped-entry check reads versions back to the newest replace".

The seed's `board` trailer is now a cut point too, so all three are stale.

**LOW — D2, `board` value in the trailer.**
D2 now writes `board` where the `--by` value goes.
D2 Bound still says "the replace that dropped the entry stays in git history with the caller's `--by`".
A seed cut has no replace or caller `--by` behind it.

**LOW — Open risks, first bullet, against D1's new Merriam bullet.**
The risk says a replace "reaches the agent only through the creel notice, as `add` does".
D1 now holds the notice back from a Merriam strand, which gets the replaced record only through a Webster re-entry.

**LOW — D1 commit-failure criterion.**
The criterion reads "while HEAD holds neither the written bytes nor its trailer".
The body's "neither" means the two cases written bytes with the trailer and written bytes without it, which together mean HEAD lacks the written bytes.
The criterion's wording names a different pair of conditions, and a reader can match it against the body two ways.

No further tool calls made.
