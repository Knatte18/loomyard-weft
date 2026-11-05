---
round: 2
exclude_lenses: []
focus:
  - "In `_lym/discussion/decision-record.md` D3, settle whether the deliver and confirm steps, taken in their final order, are exactly-once and consistent with D2's return rules: check the duplicate check against the store under the receiving daemon's lock, the confirm answer against an outbound message already returned undelivered, and the confirm tuple against every field the deliver carries."
  - "In `internal/creelengine/server.go` (request resolution before dispatch) and `internal/creelengine/identity.go` (`resolveSender`), settle whether the record states how the deliver and confirm requests from another hub's daemon pass local sender resolution, and how any answer other than accepted or refused is classified by the sending daemon."
  - "In `decision-record.md` D5 and the D3 bound, settle whether every route by which a same-user process can grant trust or forge a sender is either stopped or named as another note's to close, and whether D5's rationale claims nothing the on-disk read at arrival contradicts."
  - "In `decision-record.md` D6 and D7, settle whether every output a strand reads mail through, including the text output of `lym creel read` in `internal/creelcli/cli.go` (`writeMessagesText`), carries the trust stamp D7's instructions tell the agent to read."
---

Walkthrough and the other lenses stay on: round 1's fixes reshape D3's steps, and no lens's area is settled.
