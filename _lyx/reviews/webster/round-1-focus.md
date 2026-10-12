---
round: 1
focus:
  - "Pause sequence (card 8) and orch watcher (card 9): settle spacing across ticks. Does any path send a second Esc before the provider's settle has elapsed, or open the rewind picker?"
  - "Guarded interrupt in shuttle (card 4) and Inject refusal (card 5): is every interrupt-key path guarded by the pane reading, and does the picker close happen at most once?"
  - "Error semantics: does every caller treat `ErrNoRunningTurn` as success only where the plan says, and is `ErrRewindPickerOpen` never wrapped in `ErrSessionBusy`?"
  - "Test economy: do the tests in cards 3, 7, 8 and 9 duplicate coverage of the public-surface tests, and is each added behavior covered?"
  - "Docs and PATTERN: do `PATTERN-guarded-interrupt`, the package `doc.go` files and the CLI help match the behavior the diff builds?"
---

Seed focus from the plan's card index; the reviewer still reads the whole committed diff.
