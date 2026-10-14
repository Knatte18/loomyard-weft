---
round: 2
focus:
  - "In `_lyx/discussion/decision-record.md`, Decision 6 (confirmation and pre-hibernate check): settle whether the session states it confirms over are exactly the names `tenderengine.Status` reports (`shuttleengine.SessionStateName`), whether every state including `unknown` and the orch's reading is either withholding or explicitly accepted, and whether confirmation and the pre-hibernate check each require a live hub-scope pause entry when the daemon owns no entry (Decision 7)."
  - "In Decision 10 (sleep-command deny) and its acceptance criterion: settle whether the denied commands are real verbs of the tools named (check `systemctl --help` and `loginctl --help` on this machine), and whether the Decision states how the deny matches (command position, flags between command and verb) and what passes, as the attack-surface rule requires for a bypassable guard; compare with how `internal/shuttleengine/claudeengine/settings.go` documents its existing denies."
  - "In Decisions 3 and 5: settle whether the hidden `lyx hibernate watch` verb, which carries the daemon's effects, has a stated caller check and a stated behavior when started with no request or beside a live daemon, and whether an acceptance criterion pins it."
  - "In Decisions 4 and 8: settle whether the test for a live versus dead daemon is named and consistent with the fields the request file records."
  - "Across the Acceptance criteria section: settle whether each refusal and terminal path stated in the Decisions has a criterion a test could pin, and whether any criterion asserts behavior no Decision states."
---

The round-1 fixes are unreviewed text, so these entries re-open the areas they touched.
