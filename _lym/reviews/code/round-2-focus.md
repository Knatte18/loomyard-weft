---
round: 2
focus:
  - "internal/burlerengine/assist.go and internal/loomcli/wiring.go: does any role name or directory name (the burler review role prefix, the burler directory) stand as a repeated literal rather than one declared constant referenced everywhere, per PATTERN-named-constants?"
  - "internal/burlerengine/assist.go assistSeat and composeAssist: when composition fails (unknown fan, instruction directory not creatable, step stencil without an engine block), does Review still return a nil error with only the chair started and the assistant's end marker reading not started, given that seatengine's Table.Validate refuses blank seat values before Run reaches the skip branch?"
  - "internal/loomengine/review.go CapAssistTimeout and config load: can an assist_reviewer_timeout_min of 0 reach Seat.Timeout, where seatSpec reads a zero seat timeout as the chair's own, letting the assistant outlast the chair's timeout?"
  - "internal/loomcli/wiring.go await-cap resolution through the keep-warm resolver: does a test make the cap bind, so an ignored or wrong cap fails it?"
  - "docs/overview.md: do the loom.yaml and tapestry.yaml key descriptions and the per-perch fan sentence list every assist reviewer model, fan and timeout key the diff adds to the config templates?"
---

Round 1 covered the diff in full; this focus directs round 2 to the sites where gating findings stood.
