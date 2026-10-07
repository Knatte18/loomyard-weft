# Plan checks let a card command reach the fabric repo

`plan-fabric-reference` · Note · bug, webster, loom

Seen 2026-10-07 in card-batching: card 08 (`_lyx/plan/08-calibrate-template.md:16`) ordered `go run ./tools/tokencount ... -history ../card-batching-weft`. The fork obeyed, record-batch failed the audit with `fabric-reference`, `recover-batch` refused `needs_fresh` and `accept-audit` refused too, leaving only a full `reset --to start`.

The plan validator should refuse a card command that names a path outside the tracked tree (the weft repo above all), so Plan-Review never passes one. Separately, a `fabric-reference` from a batch with no commit and HEAD unchanged cannot have touched the tree; `accept-audit` could accept it.