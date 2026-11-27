---
round: 1
focus:
  - "Assertion preservation in `scopes_integration_test.go` and `test_integration_test.go`: pair every row and assertion at the base with its counterpart at HEAD, including assertions that sat inside removed `t.Cleanup` blocks, `defer`s, restore steps and green-record removals. Is any dropped, weakened or made conditional?"
  - "Starting state per row: for each row that used to inherit config, files, commits, directories or green records from earlier rows, does its own fixture or priming run give the state its name describes? Can any assertion, especially an absence or refusal check, now pass vacuously on a fresh hub?"
  - "Parallel safety: do parallel rows or nested rows still share mutable state, such as package variables, environment, the shared-build directory, binary or cache paths, ports or locks, captured loop or outer variables, or a parent's `t`, `scopesHub` or `scenarioHub` reached through a method or closure?"
  - "Refusals table: does each row break and repair its own hub so the slot check still observes what it did at the base, with `setup` and `wantError` bound to the same fixture?"
  - "Tier0 item of the amended entry: no case moved. Check against the code at HEAD whether any scope computation case in these tests needs no subprocess, and whether the reasons the change description gives hold."
  - "Change description against the diff and the amended entry: the measurement pairs and the command used compared with the `Added after Discussion` entry, the claims about timings, test counts and what is untouched, and the `two-way` merge door."
  - "Doc and inline comments on the new fixture types, constructors, methods and moved helpers: do they describe the per-row fixtures accurately, with no leftover wording about a shared hub or row order?"
---

The change touches two integration test files and restructures sequential rows into parallel ones.
The main risks are lost assertions, rows whose starting state changed, and state still shared between parallel rows.
