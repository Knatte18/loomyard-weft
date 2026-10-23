---
round: 1
focus:
  - "Gate routing in `NewVerifyGate`: does the cold route fire on the last re-prompted failure, after a fork that added no commit above the pre-fix head, and on every failure after a cold one, and is the commit-range comparison correct?"
  - "`lyx webster fix-verify`: are the refusal paths (no gate report, engine cannot audit forks, no `round` list, covering pause) and every terminal result (`done`, `dead` with `asking`/`timeout`/`died`) handled, with the strand stopped and the run dir removed only on `done`?"
  - "`State.VerifyFixers` lifecycle: are spawns recorded, fixer sessions added to the write evidence, stopped at run entry when not terminal and by `reset --to start`, and does a state without `verifyFixers` still load?"
  - "Prompt stencils: do the split body and the fork and cold prefixes keep every rule the old `webster-body-verify-fix` carried, and does the fixer never write under `_lyx`?"
  - "Scope against the board entry: does the `verify_gate_attempts` template drop to 2 and the reuse of the `recovery` model key fit the entry, and does the diff license nothing beyond it?"
  - "Docs and claims: do the `doc.go` sections, specs and `docs/overview.md` land in the same diff, and does each claim in `_lyx/landing/summary.md` hold against the diff?"
  - "Test economy: is each new behavior covered once, with no duplicate of an existing test and no assertion on stencil prose?"
---

Seed focus from the summary and the board entry; the reviewer still reads the whole diff as the base.
