---
round: 1
focus:
  - "D4 and D6, against the code: does Stop, the step-stop sweep and strand removal really consult no engine today, and does every other engine-consulting `Runner` operation have a record to resolve from? Settle whether the 'no shuttle run record uses the default' rule covers every such operation."
  - "D7 and D8 together: Webster resolves the default engine and asserts the fork-audit capability, while `Start` resolves the spec's engine and checks `forks`. Settle whether a Webster run whose spec names a non-default engine can end up audited by an engine other than the one that ran it, and whether the two refusals are consistent."
  - "D8 attack surface: `Supports` absent means meets none, and `New` checks `forks` against fork audit but not `peer-messaging` or `background-wake`. Settle whether any Decision introduces a skip or bypass without a stated bound, and whether the closed `Requirement` set and an unknown requirement string are bounded at every entry point."
  - "D13 and D1: the scan is worded 'any package under `internal/shuttleengine/`'. Settle whether that covers every package the allowlists drop (`reedengine`, `claudeengine`) and whether `shuttlefake` and other non-provider subpackages are handled, so the scan neither misses a concrete engine nor fails the tree."
  - "D2 and D3 against D5 and R2: the default name is fixed per `EngineSet` while records and long-lived holders outlive it. Settle whether an absent-`engine` record can be driven by the wrong engine, and whether R2's acceptance is a stated bound or an unexamined premise."
  - "Acceptance criteria against Decisions: settle whether D9's capability-missing path, D10, D11's `effort`/`version` read from `Params`, and D13's scan each have a must-cover scenario, and whether any scenario names a test to write instead of a behavior."
  - "Relocation check: look for deliberation narrative inside Decisions or Constraints that belongs in `support-log.md`; before proposing a move, extract any requirement the plan writer needs into the record."
---

Seed focus for round 1.
Entries steer attention only; the rubric's `Do not flag` list still governs, and none of these entries asks for rejected alternatives, a plan-writer notes subsection or exhaustive call-site enumeration.
