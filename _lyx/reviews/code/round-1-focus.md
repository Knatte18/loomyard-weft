---
round: 1
focus:
  - "PATTERN.md and pattern/: the plan's PATTERN changes declare adds for PATTERN-dobby-starts-runs and PATTERN-shed-stop-chain and a change to PATTERN-owner-machine; check each lands with the declared text, since the diff's pattern/ stat shows no new entry files."
  - "internal/dobbyengine, internal/beamcli, internal/hubgeom (cards 4, 27, 32 to 39): does dobby call no step body, take every path as told without importing internal/lyxcwd, keep its lock, state and markers under .lyx/beam/ named only in internal/lyxdirs, and log a yield at Debug?"
  - "Batten pass semantics (cards 13 to 16, 26, 28, 30, 31): one check per pass with the memory record persisting the former in-memory latches; the spawn-hold yield, the respawn cap and the API-error revive each bounded and idempotent across restarts; exit codes shared from the package battencli imports."
  - "Yield path (cards 8 to 10): a yielded shed step persists nothing and ends the pass without continuing the loop or deciding; check internal/shedengine, internal/shedverbs and internal/sheddriver for a path that still persists or loops on a yield."
  - "Arbiter move (cards 19 to 25): internal/shedarbiter and the sheddriver resolver against the old loomengine behavior; loom, darn and batten overrides, the darn.yaml key copy as a hubreconcile migration, and batten's eligibility rule; also check that batten does not link tree-sitter."
  - "Draft status (cards 5 to 7): the draft rule at the board's commit point in boardCriticalSection, the migration recording a key rather than raising the store version, and the cancel, claim-settlement and start-guard writers."
  - "Notices (cards 1, 2, 17, 18, 41): the landed notice and ready snapshot are dropped, the Land row still posts one line when the prime's records side did not pull, failure notices follow the stop chain, and Finalize's config notice goes key by key."
  - "Test economy: new tests that duplicate existing coverage, and behaviors the diff adds with no test."
---

Seed focus from the plan overview, the decision record's constraints and the review range's file stat.
