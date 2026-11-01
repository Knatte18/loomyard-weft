```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked whether the round's changes still agree with the sibling cards, criteria and cross-references they touch.

1. **LOW.** Card 10 (fabric-worktree-seam) now fires "took in" from `Fabric.FastForwardPair`, but two sibling lines still describe the old reach.
   - `00-overview.md`'s Card Index line 10 says the card fires the seam "from add, clone, pull and origin reconcile".
   - Card 10's ImpactSummary says it "Adds two optional seam calls to fabric's add, clone, pull and origin-reconcile code moves".
   - Neither mentions the parent-pair fast-forward the new bullet and test row add.

2. **LOW.** In card 7 (module-bolt-dir-uses), the bullet on `hubforge`'s `Hub.BoardDir` ends "its callers' tests being card 6's". Card 6 does not cover those callers.
   - Card 6's Edit list names only `internal/hubforge#assertRealHub` from hubforge.
   - Card 6's Intent moves only `"_board"` strings and hand joins to the constant. It never touches callers of `Hub.BoardDir`.
   - So the cross-reference points at work no card does.

3. **LOW.** Card 2's ImpactSummary is stale. It still lists only the value-neutral constant moves and the geometry guard's narrowed row.
   - The card's Edit list and Intent now also change `internal/runproc#allowedImports` and the `leaf_enforcement_test.go` header.
   - That is a second guard's allowlist growing, which a reader of the summary would not see.

Everything else agrees. The renumbered card references match the new numbering, including the overview's "Applies to" lines and the mentions in cards 3, 4, 5, 9, 10, 11, 12, 14, 15, 16, 17 and 18. `ExcludeWayForward` is consistent across card 9's Create, card 15's Uses and the shared decision. The new Reconcile row in card 9 matches its new bullet. The trimmed `TestTable` matches card 12's unchanged `TestFabricReplay`. The TestMain additions match PATTERN-test-isolation.

Further tool calls: none.
