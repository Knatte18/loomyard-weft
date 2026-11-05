---
round: 3
focus:
  - "In _lym/plan/09-slackengine-relay.md, the whileAway restart cap kept in RelayState StartFailures, NextStartAt and LastSpawnAt: settle who writes each field, in what order relative to the spawn call, what happens when spawn returns an error before any relay exists, and which writer resets StartFailures after an accepted first read, then check that a TestEnsureRelay row reaches each path."
  - "In _lym/plan/03-creel-seams.md, every doc comment or doc.go sentence in internal/creelcli and internal/creelengine that ReadWait (a caller-told timeout) or HubStrands makes false: settle whether each such ref sits under Edit rather than Extends."
  - "In _lym/plan/07-slackengine-inbound.md, step 3 of the inbound poll: settle the outcome for each error Hub.Paused or Hub.Beams can return, relative to whether the creel post already succeeded and whether the cursor moves, so that each message is handled once."
  - "In _lym/plan/10-slack-cli.md, the cmd/lym mount of the slack module: settle whether every text that cmd/lym/clitree_test.go checks against the mounted modules, including newRoot's Long list, is carried by a card target."
---

Round 2 has one BLOCKING `design` finding on the restart cap, so the first entry names it.
The other entries name the places where round 2 found card text that left an outcome unassigned or an Extends ref falsified.
