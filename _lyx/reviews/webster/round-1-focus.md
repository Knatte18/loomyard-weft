---
round: 1
focus:
  - "configengine machine layer (card 1): does the layer apply only to a hub-wide spec with a `machine` key, judged on told `Dirs` with both `Board` and `Anchor` set, and does every hub-file check on `Dirs{Board}` skip it?"
  - "Gate load failures (card 2): does every failure path name both files and both ways forward, while an absent hub file stays `ErrConfigAbsent` whether or not a machine file exists?"
  - "`lyx config --layer machine` (card 3): are print, set, editor, menu marks and `--resolved` guarded like the default layer, and does set validate through `ValidateMachine`?"
  - "Card 4: does the `hubreconcile.Ensure` test pin a byte-for-byte unchanged machine file, and does the diff leave `configsync` and `hubreconcile` code untouched?"
  - "PATTERN declared changes: does the diff land `PATTERN-config-layers` and `PATTERN-hub-wide-config` with the declared text?"
  - "Test economy: do the new tests in the four cards duplicate each other's coverage, and does any added behavior lack a test?"
---

Focus follows the plan's shared decisions and declared PATTERN changes.
