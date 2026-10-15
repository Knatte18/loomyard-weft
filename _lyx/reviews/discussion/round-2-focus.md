---
round: 2
focus:
  - "In decision-record.md, check the D1 hint, D4's write rule and the Acceptance criteria together: after a plain fresh launch that finds a recorded session id, does the id the hint names still identify the conversation the operator wants back, given that the launch saves its own new id? Settle where the previous id is read relative to that save (`internal/orchcli/start.go`)."
  - "In decision-record.md, check D4, D3's Bound and the Scope Out bullet on `lyx reed resume` against every route that writes `session_id`: fresh launch, adopt, the watcher's tick, `start` binding a live strand without launching, and a strand reed relaunched with its stored resume command. For each, does the record state which id lands and what bounds it, and does every claim about what `--continue` recovers hold?"
  - "In decision-record.md, check that every added or changed hint, field and Open risks bullet states what it can let through and what bounds it, and that two hints competing for the envelope's `hint` key are given a stated order."
---

Round 1 changed the id-write and hint decisions; this round reads the revised text fresh against those two questions.
