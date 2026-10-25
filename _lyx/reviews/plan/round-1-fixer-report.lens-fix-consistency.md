```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fixer's change set agrees with its sibling cards, decisions and cross-references (fix-consistency lens).

**Findings**

1. **LOW: card 18's Intent conflicts with D6's wording on when the create record is removed.**
   - Changed text: card 18 now says Add "removes the move record before it returns, after those steps, so ahead of `createSteps`' push, workspace and dobby steps".
   - Evidence: D6 in `decision-record.md:93` still says Add "removes the record once the remaining create steps are done".
   - The change is sound, since card 16's gate would otherwise refuse the create's own dobby start. But no Added after Discussion entry and no sentence in the card ties the earlier removal back to D6. A reader comparing card and decision sees the two disagree.

2. **LOW: card 18's `TestAdd_AtAnchor` row was left stale.**
   - Changed text: the same card 18 Intent sentence now places the workspace step in `createSteps`, after `Topology.add` returns.
   - Evidence: the unchanged `TestAdd_AtAnchor` row, an engine-level `Topology.Add` test, still asserts that a beam added with `Anchor: "y"` has "launcher and workspace folder at `y`". The workspace regeneration that row expects is now stated to run outside Add.

3. **LOW: two headings don't mention the fast-forward weft pull that cards 9 and 13 now cover.**
   - Changed text: card 9's `Fabric.fastForwardWeft` paragraph and card 13's `fastForwardWeft` caller line.
   - Evidence: the overview's Card Index line for card 9 still reads "the one comparison, the pull stop and the replaying-push stop". Card 13's title still reads "A standing move record stops Add, reconcile, pull, push and the origin reconcile". Card 13's ImpactSummary was updated to "seven weft writers, checkout and the fast-forward's weft pull included", so its title and summary now disagree.

4. **LOW: card 18's recovery path ignores the result `Apply` now returns.**
   - Changed text: card 18's `EnsureAnchor` now "recovers through `BeginAnchorChange` with the record's `B`, `Apply` and `Finish`". Card 15 re-signed `Apply` to return `(landed bool, err error)`.
   - Evidence: card 17 handles `landed` true and false separately. Card 18 doesn't say what `EnsureAnchor` does with `landed` or how an after-commit failure surfaces to `createSteps`. Card 15's after-commit error names `lyx fabric anchor &lt;slug&gt; --set &lt;B&gt;` as the re-run, while card 18's re-run is `lyx beam create`, so the create path names a different re-run than its own.

Tool calls:
- Grepped `decision-record.md` for failure, restore and `--anchor` text, because my context held only D10 and the Added after Discussion entries, not D6 or D7's step text.
- Read D6 (lines 89–97) and D7 step 4 (lines 140–150), to compare them with the changed card 15, 17 and 18 text.
