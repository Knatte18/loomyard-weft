---
round: 1
focus:
  - "D2: does the lockless reader's claim of atomic writes hold for every writer of the orch state file, and is the two-strike tolerance stated the same way in D2, the acceptance criteria and the ledger?"
  - "D2 and D4: the 'absent file reads as not found' path against the 'state unreadable' path; is a decode failure of a present file (not a stat or read failure) assigned to one outcome?"
  - "D3: is the stale-threshold bound consistent with the claim that the runs strand is replaced only on a build change or a dead strand?"
  - "D4: does every rule in the line's content have a matching acceptance criterion, and does any rule (phase during a cycle, stopped session) state something the persisted state cannot back?"
  - "Decisions against Constraints: does any PATTERN constraint cited contradict a Decision, such as the log level or the config strictness reading of `LoadConfig`?"
  - "Support log against decision record: does each Turn 3 'fixed' line show up in the record, and does the record carry any requirement that sits only in the support log?"
---

Seed focus for round 1.
Entries target the places where a Decision's stated premise or bound could disagree with another section of the pair.
