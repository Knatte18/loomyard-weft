# Plan checks let a card command reach the fabric repo

`plan-fabric-reference` · Note · bug, webster, loom

Seen 2026-10-07 in card-batching: card 08 (`_lyx/plan/08-calibrate-template.md:16`) ordered `go run ./tools/tokencount ... -history ../card-batching-weft`. The fork obeyed, record-batch failed the audit with `fabric-reference`, `recover-batch` refused `needs_fresh` and `accept-audit` refused too, leaving only a full `reset --to start`.

The plan validator should refuse a card command that names a path outside the tracked tree (the weft repo above all), so Plan-Review never passes one. Separately, a `fabric-reference` from a batch with no commit and HEAD unchanged cannot have touched the tree; `accept-audit` could accept it.
The no-commit case has since landed (`accept-audit --batch`, 83a7d6691).

Seen again 2026-10-07 in shuttle-turns, batch 13 (cards 13–16): a fork, most likely for `15-replay-corpus`, read other worktrees' shuttle events (`*/.lyx/shuttle/*/events.jsonl`, `*-weft/.lyx/shuttle/...`) with `cat`/`jq` to build a corpus, then committed all four cards.
The findings were read-only and pathless, but the batch had commits, so `accept-audit --batch` refused and the hub had to `git reset --keep` the batch's start and redo all four cards.
Two wants:

- `accept-audit --batch` accepts pathless `fabric-reference` findings on a batch with commits when the findings come from read-only commands, or the audit tells a read from a write in the first place.
- The plan stencil steers a corpus or fixture card to committed test data, never to live files of other worktrees.
