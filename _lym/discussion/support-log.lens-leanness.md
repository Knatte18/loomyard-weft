```yaml
lens: leanness
outcome: reported
```

Lens: leanness. I looked for content in the decision record that should move or be cut. Tool calls: none, since my context already held both files and the rubric.

**Findings**

- **MEDIUM — `## Open risks`:** The only risk is a known limitation the design accepts, not a question for the parent.
  - The line says an older-build session's lost checklist "comes back only on the next `lym loom start`".
  - Under the Step 4 and rubric `decision` meaning, Open risks is where questions go up to the parent. This line asks nothing, so the Discussion-Review perch would raise a non-question.
  - Fix: move it to `## Scope` Out ("Restoring a checklist an older build already dropped; the next `lym loom start` restores it") and leave Open risks empty or with a "none" line.
- **LOW — `## Scope` In:** Each "In" bullet restates D1–D4 almost word for word: shared spec, move in `ensureRunsStrand`, `Display.Title` naming, overview line. The only bullet that adds anything is the docs line, and the acceptance criteria's Review bullet already covers it.
  - Fix: cut the In list down to one line pointing at D1–D4 plus the docs. Keep the Out list, which carries real exclusions.
- **LOW — `## Auto-mode assumptions`, third bullet:** It says "The fix goes in the runs strand's replace path, not in a reordered bring-up (see the support log's rejected alternatives)".
  - This duplicates the second Scope Out bullet and D2.
  - It also points the plan writer at the support log, which the plan writer never reads.
  - Fix: cut it. The ledger's auto-pick Q1 already records the choice.
- **LOW — `## Goal`, second sentence:** The "Today `ensureRunsStrand` replaces…" sentence is cause narrative that D2's rationale already carries ("the replace is the one place the checklist is lost").
  - Fix: cut it from Goal, or keep it and trim D2's rationale. It should be said once.
- **LOW — D2 rationale, second sentence:** "Writing the sidecar keeps the moved strand from being relaunched…" repeats step 2's own "so the next `lym loom start` keeps the strand".
  - Fix: cut it.
- **LOW — `## Constraints`:** The `PATTERN-named-constants`, `PATTERN-loom-contract` and `PATTERN-test-speed` bullets restate repo-wide rules without tying them to anything specific in this design, unlike the shed-driver and sidecar bullets.
  - `PATTERN-loom-contract` ("no loom verb branches on a recipe's name") is already covered by the `PATTERN-no-hard-coded-choice` bullet.
  - Fix: drop the loom-contract bullet. Keep test-speed only for its specific tier note.
- **LOW — `## Notes for the plan writer`:** The first bullet names the same files `relevant.md` lists (`spawn.go`, `ownwindow.go`). Its only extra is the `overviewWindowTitle` member. The second bullet also overlaps `relevant.md`'s `overview.go` line.
  - Fix: put `overviewWindowTitle` on `relevant.md`'s `spawn.go` line and cut the Notes section.

**Other (outside the lens)**

- **MEDIUM — D4 and D3:** When the seed is unreadable, D3 leaves the window named by its role, `loom-status`. D4 says nothing about what the overview line shows in that case.
  - D4 also draws no line when there is no seed, so in that case there is nothing to point at. The case falls through, and it may only need saying once.
  - This belongs to the walkthrough lens; I'm noting it only.
