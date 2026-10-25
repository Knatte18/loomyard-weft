---
round: 2
focus:
  - "Cards 28 (StartFirstPass) and 33 (SpawnPass) start a detached `<lyx> batten run <slug>`: settle from the cards' Intents which working directory each pass gets, and whether batten's pre-run (`resolvePersistentPreRun` via `lyxcwd.Resolve`) can resolve a location there, given the hub root is not a git repository."
  - "Cards 13, 28, 4, 35 and 36 change production code (InnerRun yielding after one check, removed `windowFlag`/`openTerminalFlag`, the closed import allowlists for `battenshed` and `dobbyengine`): settle whether each card lists the tests, callers and allowlist entries its change forces outside its own package."
  - "Card 13 (`childWait.begun` in InnerRun's `Call`): settle at which point `begun` is read relative to `observeState`, and whether a child that goes running to done between two passes still produces a state-change Stuck before the Done arm."
  - "Card 25 (batten arbiter wiring, no goto guard) and card 4 (dobby `ReadState`/`UpdateState`): settle what bound each card's Intent states for the new goto edge, and what a pass or tick does when the dobby state record does not decode."
  - "Cards 24 and 18 (arbiter eligibility for a batten stop, pause-honoured Land stop): settle whether a blocked pause-honoured stop can reach the arbiter, and whether the landing-halt test row can be reached through `Check`."
---

Round 2 re-reviews the plan after the round 1 fixes.
