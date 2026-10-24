```yaml
lens: fix-proposal
outcome: reported
```

Task: fix-proposal lens, challenging the reviewer's proposals the fixer carried out as proposed.

**[BLOCKING] F1's move does not keep every card gate under the headroom line**

- **Changed text:** `_lyx/plan/03-pattern-shorten.md` (card 23 moved to position 3), together with the PATTERN growth on cards 4, 8, 20 and 32. Card 32 is the card that now holds the old card 33's `PATTERN-tier-tags` line.
- **Summary:** Even with the shortening first, the last growth card's `lint-pattern` step pushes the file past `HeadroomBytes` (15564), so card 32's gate still fails.
- **Evidence:** I measured the line bytes from `PATTERN.md` and the plan cards:

  | Step | Change (bytes) | PATTERN.md size |
  |---|---|---|
  | HEAD today | — | 15651 |
  | Card 3 shortens 19 lines (3033 → 2551) | −482 | 15169 |
  | Card 4 (`PATTERN-test-tiers` 134 → 203) | +69 | 15238 |
  | Card 8 (`PATTERN-gate-carry`) | +122 | 15360 |
  | Card 20 (`PATTERN-row-entry-gate`) | +133 | 15493 |
  | Card 32 (`PATTERN-tier-tags`) | +137 | **15630** |

  15630 is larger than the HEAD it compares against (15493) and larger than 15564. So `pattern.Headroom` reports at card 32's gate, the same rule the review cites at `internal/pattern/headroom.go:12`.
- **Why the proposal is wrong:** It rests on "each intermediate size stays at or below the final projection that pattern-headroom already admits". But the plan check compares the final projection against the branch base, 15651. 15630 passes that check while still being above the 15564 headroom line. A single card gate is judged against the headroom line, not the base.
- **Size of the gap:** Card 32 needs at least 66 more bytes cut, done before card 32 runs. That could be one more line shortened on card 3, or a shorter `PATTERN-tier-tags` line.

Tool calls beyond my context:
1. A byte-count script that failed on a `tr` range error; I re-ran it as call 2. It read the new PATTERN lines' sizes from cards 8, 20 and 32 and the old and new `PATTERN-test-tiers` line sizes, which my context held only as the review's figures.
2. Summed the current `PATTERN.md` bytes of the 19 lines card 3 rewrites against card 3's new text, to get the cut, which my context did not hold.
