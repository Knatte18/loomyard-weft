---
round: 1
focus:
  - "Decision 8 and Decision 4 (cancel, and a replacing request, over a dead daemon's request): does each state what it can now lift or skip, and is the liveness test for a 'dead daemon' (pid plus start time) named so a reused pid cannot make a live daemon read as dead or the reverse?"
  - "Decision 6 (confirmation and the pre-hibernate check): does the window between the last check and `systemctl hibernate` have a stated bound, and does the confirmation list leave a session state tender can report that is neither covered nor excluded?"
  - "Decision 7 (owned entry by identity, lift on wake): can the lift-by-identity lift an entry the operator re-added with the same scope, caller and paused-at time, and is the five-minute wake window's failure path bounded for a hub left paused?"
  - "Decision 2 (`--after` firing condition 'its current row is past Land'): is the condition derivable from the batten status fields the decision names, without the plan writer having to invent a row ordering?"
  - "Decision 10 (sleep deny): is the bound exact about what the deny matches, including how a command prefixed or wrapped in another command (`sudo`, `env`, a shell `-c`) is treated, and what that leaves bypassable?"
  - "Decision 3 (caller check): the stated bound is a guard against a mistaken call; does any verb or flag let an agent reach the daemon's effects some other way?"
  - "Acceptance criteria against Decisions: does each Decision with a bound or a refusal have a criterion that a test could pin, and does any criterion assert behavior no Decision states?"
  - "Relocation: any deliberation narrative left in `decision-record.md` Decisions or Open risks that belongs in `support-log.md`, after first extracting any requirement it carries into a Decision or Constraint."
---

Seed pass over both files and the writer-side spec.
Not raised: a missing "Notes for the plan writer" subsection, absent rejected alternatives in the record, and incomplete call-site enumeration, all excluded by the rubric.
