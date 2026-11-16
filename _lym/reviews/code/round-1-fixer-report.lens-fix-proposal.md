```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: challenge the reviewer's proposals the fixer carried out as proposed.

**F-A (MEDIUM)** — `internal/retroengine/retro.go`, the cap branch in `Select` (reviewer's F6 proposal: drop kept runs tied with the first left-out run, keep at least one).
- **Summary:** when the tie reaches the first kept run, the proposal widens the loss instead of closing it.
- **Evidence:** take `MaxRuns: 2` and three runs landed in the same second `t`, `a`, `b` and `c`, ordered by slug.
  - The cut keeps `[a, b]`, and `firstLeftOut` is `c`'s landing `t`.
  - The loop drops `b` because `b.Landed == t`, then stops on the `len(runs) > 1` guard.
  - The plan is `[a]` with `until = t`.
  - The next retro keeps only runs landed strictly after `t`, so `b` and `c` are both lost. Before the fix only `c` was lost.
- **Why the fix is wrong here:** the guard keeps one run, but that run is still tied with the left-out ones. The same happens whenever the first kept run shares the second with the cut, such as `[x@10, y@10, z@10]`.
- **Not tested:** the added `TestSelect` row has a tie that starts after the first kept run (`x@8`, then `y` and `z` at 10:00), so the degenerate case is not pinned.
- **Fix to consider:** when the tie reaches the first kept run, extend the kept set through the whole tied second, which exceeds the cap, or refuse with a message to raise `--max-runs`.

**F-B (LOW)** — `internal/retroengine/retro.go`, the named-run refusal (reviewer's F12 proposal).
- **Summary:** the message now blames the lookback for every unlanded name.
- **Evidence:** `newest[slug]` is missing for a typo or a never-landed run just as for a run landed 70 days ago. The text "has no landing ... within the last 60 days ... or cover an older run with --since and --until" points a typo at a window flag. The earlier "name only landed runs" fit both causes.
- **Fix to consider:** word it to cover both causes, for example "no landing within the last 60 days (a typo, an unlanded run, or an older run)".

Proposals I challenged and found sound:
- F1: the `ls-tree -- "$USAGE_DIR"` and `show "$tag:$SHED_DIR/..."` forms are equivalent to the old text.
- F2: `--until <today>` parses to midnight, which is never after `now`, so it is still accepted.
- F7: the new wording holds for both stamp kinds.
- F8: the slug uses `until`, which is unchanged, and only `Body` reads `From`.
- F9, F10, F11, F13, F14: the added assertions match the fixture arithmetic. The gate was green.

Further tool calls: none, because my context already held every file I needed.
