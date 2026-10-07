---
round: 2
focus:
  - "In `_lyx/discussion/decision-record.md`, check D3 against every route by which fabric hands the prime's code side to a push: `PushPairAnchored`, the `sync` verb's `spawnPush`, and `Fabric.Commit` in `internal/fabricengine/commit.go`, which ends with `spawnDetachedPushFn` whenever a side committed and runs `lyx fabric push` as a child. Question: does the Decision state that the code side is dropped at each route, how a route that holds only paths learns it runs in the prime, and what bounds the change?"
  - "In `_lyx/discussion/decision-record.md`, check D2's rollback deletion condition against `probeWarpLeftover`, which returns an empty adopt tip both when the origin branch is absent and when a pre-existing origin branch sits at or behind the HEAD the new branch forks from. Question: can the rollback delete an origin branch that existed before the `Add` call, and does the Decision state what its guard lets through and what bounds it?"
  - "In `_lyx/discussion/decision-record.md`, check that the Constraints bullet on `PATTERN-mutation-record` agrees with `pattern/PATTERN-mutation-record.md` (an executor appends its primitive only after it observably changed state; fixed envelope keys `mutations` and `partial`) and with D4's renamed verb fields. Question: does the bullet leave any reading under which a plan writer records a refused push attempt or treats D4's renames as forbidden?"
  - "In `_lyx/discussion/decision-record.md`, check D1's retry classifier and Bound. Question: is there one classifier rule for a stderr that mixes `remote:` lines with a `[remote rejected] <ref> (failed)` line, and does D2 say how an exhausted retry on an ambiguous remote state resolves?"
---

Focus entries target the open design questions and the mismatches between Constraints and Decisions.
Acceptance-criteria coverage, the hard-cut assumption against persisted traces in `contracts/stencils/shed/shed-template-driver.md`, and relocation of narrative to the support log remain in scope under the normal rubric.
