---
round: 1
focus:
  - "Open risks, parent commit `7c23591`: check against the repository whether it is landed or contradicts D1 to D3, and settle whether the question is a `decision` finding or one the record's own wording already answers."
  - "D2 and D3 attack surface: check that every bypass or skip they add (replace of `loom-status`, fall-back to today's replace, a reserved or empty `Title` falling back to the role name) states what it can let through and what bounds it."
  - "D2 case 1 and case 2 and the failure bullets: check that each outcome is stated for every strand state (none, live, dead, in its own window) and that no failure path leaves a state the criteria do not name."
  - "D3 and the `reedengine` claims (reed names an own window by role, reads `Display.Title` only for the overview, `StrandStatus` carries no command or display): check each rests on a stated source, and that no tmux or reed behaviour the design depends on is left for Plan to find out, such as two strands sharing one `Title`."
  - "Acceptance criteria against D1 to D4: check every Decision id is named, each criterion is behavior a test can check or is marked `Review:`, and the D4 criterion is checkable without a live reed."
  - "Relocation test for `decision-record.md`: look for deliberation narrative in Rationale or Open risks that belongs in `support-log.md`, extracting any requirement it carries into Decisions or Constraints first."
---

Seed focus for round 1.
Rejected alternatives and a missing Notes section are by design and are not flagged.
