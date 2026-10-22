---
round: 1
focus:
  - "PATTERN declared changes: check that `PATTERN-verified-tree` lands the declared text verbatim, and that `PATTERN-pause-holds-prompts` lands the departure the plan's Shared Decisions describe (summary line in PATTERN.md, declared text in full in the background file), including the eight trimmed entries in card 8."
  - "Verify sites: every call to `verifytree.Verify` (loomshed, webster, `lyx webster verify`, Merge-In, Finalize, Publish) is told a pause hold from `verifytree.NewPauseHold`; settle whether any site passes a nil or empty hold, and whether each site's drift-guard test pins its wiring."
  - "Shuttle send refusal: `ErrPaused` and `PausedError` are judged by the target's own slug after the idle wait; settle whether every production caller that types a line (run start, deferred prompt, gate loop, seat notice sender, `lyx loom resume`, `lyx shuttle send`, tender resume line, creel inject and interrupt) holds, claims or refuses as its card says, and that the orch runner stays unheld."
  - "Claim and interrupt paths: `pauserecord.ClaimSession` and the cards using it (14, 15, 16) record the session in the live entry covering the slug; settle whether `Store.interruptTarget` changes also hold for `lyx creel inject --interrupt` as the Shared Decisions state."
  - "Verify cancel: `verifyrun.Run` sends SIGTERM to the group, SIGKILL after the delay, and returns only once the group is gone; settle whether the cancel and the `paused` marker reads (card 4) cover the error and race paths the tests exercise."
  - "Test economy and commit messages: look for new tests duplicating an existing test's coverage, and for any `Pattern-Departure:` trailer that names an unknown entry or one that does not forbid the card's way."
---

Seed focus drawn from the plan's card index and Shared Decisions; no prior round exists.
