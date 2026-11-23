---
round: 1
ledger:
  - key: note-dependency-still-refused
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: remove-entry-leaves-depends-on
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: skill-and-doc-contradict-note-dependency
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: duplicate-sibling-row
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: brieftask-doc-omits-pending
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 found five findings, all local corrections whose fix text is already specified.
The two BLOCKING scope findings are the note-as-dependency refusal in `validateWrite` and the missing depends_on strip in `RemoveTask`.
The MEDIUM doc finding follows the first: the skill line and package doc must state the new rule once it lands.
