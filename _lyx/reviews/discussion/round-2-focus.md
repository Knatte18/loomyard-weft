---
round: 2
focus:
  - "D10 in decision-record.md marks the `landing` config module `HubWide` so it is read and written at `<BoardDir>/_lyx/config/`: settle whether the Decision states what stops a task-pair agent from running `lyx config set landing publish_verify=` and thereby changing it for every pair, and whether that bound names a mechanism that exists or is decided in the record."
  - "D7 in decision-record.md runs `lyx gate test` unslotted outside a hub: settle whether the Decision says which directory decides hub versus standalone (process cwd, `-C` target, or the `cliwire` target dir), and whether its Bound states what the unslotted path lets through."
  - "D1, D2, D5, D7 and D8 in decision-record.md chain one `lyx gate test` call per module inside a gate, each bounded by `cli_wait_sec` (default 300), while gate commands run under the Bash tool's maximum timeout: settle whether the record states a relation between the per-call wait, the number of chained calls and that timeout, so a Bash call is not killed mid-run."
  - "D11 and D12 in decision-record.md use a failure record under the pair's own `.lyx/` to drive the round pass and the review widening: settle whether the Decisions state what deleting or editing that record skips and what still bounds it."
---

Each entry names a Decision whose guard, routing edge or bound the attack-surface question applies to.
None asks for content the rubric's `Do not flag` list excludes.
