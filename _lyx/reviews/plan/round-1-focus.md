---
round: 1
focus:
  - "Card 5 `createOwnWindowPaneLocked`: the Intent says its three calls now pass `st` along, which changes a parameter or call shape; settle whether its member glyph needs the re-sign arrow and a signature, and likewise `markResolvedStrandWindowLocked` and `OpenWindow`."
  - "Cards 6 and 7 both edit `killStrandPanes` and `openOwnWindowLocked`: settle whether card 7's Intent changes either signature (re-sign arrow) and whether card order 5, 6, 7 holds for every `Uses` (`sessionWindow`, `windowPane`, `selectOnEnd`) against the later cards."
  - "Test economy across cards 5, 6 and 7: `TestWindowOrder`, `TestWindowOrder_RealTmux` and `TestEnsureShellWindow` gain overlapping rows (shell fix via `AttachArgv(0, 0)`, replace selection, viewed-window-ends); settle whether any new row repeats behavior an existing test or another row of the plan already covers, naming that test."
  - "Reachability of refusals and guards: card 6's one-overview-per-session refusal, card 5's `planShellWindowFix` unlisted-shell branch and card 7's failed-current-window read and failed client listing; settle that each has a named input and test that reaches it past upstream checks."
  - "Attack-surface question: the new `overview` anchor, the `select-window` appended to attach argvs, the shell rename and `move-window` on a live session, and the self-re-exec in card 1; settle that each Intent states what it can skip or let through and what bounds it."
  - "Fidelity to the decision record: settle that D1 to D10 and the Constraints each have a carrying card, in particular D10's once-per-episode logging, D5's psmux caveat, D8's never-exec-under-`go test` and Windows no-op, and D6's Alt+Down/Alt+Up hidden-window step; and that no card carries scope the record does not license."
  - "Verify coverage: settle that every package a card targets (`internal/selfexec`, `statuspane`, `reedengine` and `render`, `loomcli`, `orchcli`, `batten` and others cards name) is run by the `## verify:` commands, including tests behind the `integration` tag, and that the section is in template order."
  - "Granularity of card 5's new symbols (`sessionWindowFormat`, `parseSessionWindows`, `planShellWindowFix`, `hiddenWindowInsertArgs` as separate Create glyphs): settle whether a private supporting symbol should merge into another card entry."
---

Seed focus from a read of the overview and cards 5 to 7 plus the decision record; cards 1 to 4, 8 and 9 were not read in depth, so entries on them are left to the reviewer.
