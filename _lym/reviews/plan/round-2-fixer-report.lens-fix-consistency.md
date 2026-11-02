```yaml
lens: fix-consistency
outcome: reported
```

Fix-consistency lens report: the change set checked against the sibling cards, the overview's Card Index and Shared Decisions, and every card-number cross-reference.

No findings.

- Every bare card-number reference across the plan now matches the renumbered Card Index. The cards' own headings, the Card Index entries for cards 4 and 10, and the references updated after the insert all agree, including those in cards 1, 6, 7, 8, 9, 11, 12 and 13 and card 3's `card 14`.
- Card 3 hands its three file-header bullets to card 4 and names card 4 as their owner. `testReplyPath`, added under **Edit:**, matches its remaining comment bullet. The new card 4 `Uses` the `internal/websterengine#RecoverSpawnOrAttach` that card 3 edits, so it is ordered after card 3.
- In card 10, the reworded opening agrees with the bullet on guard order, the in-effect case, the ImpactSummary ("once shed and the settled guard admit a move") and the Card Index line ("before it writes the move"). Neither the overview nor cards 1, 2 and 14 carry a twin of the "transient" or "admitted" wording. Card 1's "wrapped as transient" is webster's way-forward wording, which the review exempts.
- In card 10, `internal/loomcli/doc.go` moved from Uses to Extends with its Intent bullet. `internal/shedengine#GotoGuard` under Edit is paired with `internal/shedengine/doc.go` under Uses and has its own Intent bullet.

Tool calls:
- grep for every `card N` and `cards N` reference in the plan's cards and overview, because my context showed only the spans the fixer edited, not the whole set after the renumber.
- grep for twins of the goto, transient and file-header wording in the overview and cards 1, 2 and 14, because my context did not hold those files.
