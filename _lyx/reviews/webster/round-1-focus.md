---
round: 1
focus:
  - "Owner check coverage: every start, resume, wake and landing path (batten, loom, InnerRun, tender, Finalize, orch start) calls the owner check before acting; does any path write or claim without it?"
  - "Owner write rule: the setter in `boardengine` is the only writer of `owner_machine` and `prime_owner`, and a takeover happens only through `OwnerRequest.Takeover` from `lyx adopt`; does any other caller move an owner?"
  - "Effective owner resolution: an entry with no owner reads as its parent's owner, a run-held entry with none reads as no other machine's, and a hub with no prime owner reads as owned by none; does the diff apply each case as the PATTERN entry states?"
  - "Finalize carve-out: a landing already past its parent-side merge finishes, while one before it stops on an owner elsewhere; does the diff draw the line at the right step?"
  - "Declared PATTERN change: `PATTERN-owner-machine` landed with the declared text verbatim in `pattern/PATTERN-owner-machine.md`, a shortened overview line, other entry lines cut without changing their rules, and a `Pattern-Departure:` trailer on card 25's commit."
  - "Machine name sourced only from `internal/machinename`; does any code read the hostname or machine name elsewhere?"
  - "Test economy on the larger cards (reconcile owner outcome, batten and loom owner checks, adopt): duplicated coverage, and behaviors added with no test."
---

Seed focus from the plan's Shared Decisions and its PATTERN change; no round has been judged yet.
