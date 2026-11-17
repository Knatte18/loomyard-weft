---
round: 3
focus:
  - "Cards 6 and 8: check that the plan's edits keep `cmd/lym`'s `testOnlyCodeAllowed` allowlist in step with production reachability once `shuttlecli` calls `shuttleparity.Report`; settle whether any entry becomes stale (for example `internal/seatengine.Prompts`) and whether a card edits `cmd/lym/testonlycode_linux_test.go`."
  - "Card 6 `(*Engine).SeatTables`: check that every seat table it builds fills every top-level marker of its stencil, since `seatengine.Specs` renders through `Prompts`, which refuses an unfilled marker; in particular compare the burler fix table's values with those `ProbeFix` in `internal/burlerengine/fix.go` passes."
  - "Cards 6 and 7 parity fixtures: settle where each seat's and producer's skills come from, how `PrepareEach` expands a spec's `ConfigRole` skills, and the on-disk skill layout Codex's `skillPath` in `internal/shuttleengine/codexengine` resolves; a fixture whose skill list is empty or laid out flat leaves the skill-load channel unexercised."
  - "Card 8 verb exit codes against the decision record's D10 and every-seat decision: settle which conditions make `lym shuttle manifest` exit non-zero, and whether a seat Codex cannot start can ever pass."
  - "Card 6 `Compare`: check that every condition its Intent gates on is computable from the inputs the signature takes."
  - "Card 2 `copyManifest`: check that each warning's way forward fits every cause its error can have, and that a test row reaches each named refusal."
---

The focus lists where to look; every other part of the plan stays in scope for the normal rubric.
