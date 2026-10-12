---
round: 1
focus:
  - "internal/state write path (cards 2-3): does every rewrite keep the old file's unknown keys, and do retired keys, concurrent writers and a missing or unparsable old file behave as D2 and D3 specify?"
  - "ReadJSONStrict to ReadJSONNoMkdir rename (card 4): confirm no caller, comment or doc still names the old identifier, and that the rename group's mechanical check ran against every target."
  - "Tolerant decoders (cards 5-6): seed.json and batten cleanup accept unknown fields, and an agreeing seed keeps its unknown keys on rewrite."
  - "board.json encoding (card 1): are the Marshalers still opaque to the new merge, and is the Save byte pin correct?"
  - "Watchdog (cards 8-11): busy_quit_sec wiring, proc-tree CPU sampling and ancestry walk, Linux-only build constraints, and SIGQUIT delivery in lyx gate test; check races and process-group handling."
  - "Gate halts (cards 12-14): webster, loom and landing read the line only through gateslot's WatchdogLine, FindWatchdogQuit and BusyQuitWayForward; webster never re-runs a hung step."
  - "Test economy: new tests duplicating an existing test's coverage, per-helper tests covered by a public-surface test, and behavior the diff adds with no coverage; test-budget.yaml updated where a package passes its row."
  - "Docs landing with code: doc.go, PATTERN.md and the PATTERN-tolerant-records scan match the behavior the diff adds."
---

Focus is set from the plan's card list; no round has been judged yet.
