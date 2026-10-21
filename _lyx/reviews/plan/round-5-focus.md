---
round: 5
focus:
  - "For every card whose Edit group changes a constructor, required reader, deps struct, func-typed seam field or skip rule, find the tests that reach it without naming the changed symbol: hand-built fixtures that bypass the production constructor, and reflection-driven seam inventories such as envkit.NilSeams, envkit's nilLegal and battencli's intentionallyNil. Settle whether each is listed in that card's Edit group or Tests paragraph."
  - "For every card that stops reading an input (a plan's `## verify:` section, a `verify` value, a `go_parallel` key, a guard scan), find the tests and helpers that still feed or assert the retired input, and the helpers left unused after a Delete. Settle whether each is carried by a card."
  - "For cards 16 and 18, which load gate.yaml at wiring and in the validate-plan verb: settle whether an absent gate.yaml (gateslot.ErrConfigAbsent) is a refusal there, and whether that agrees with card 17's Claude.Prepare, wireHub's tolerant-hub contract and the existing hub and validate-plan fixtures that seed no gate.yaml."
  - "For every Edit member glyph whose Intent changes parameters or results, settle whether the re-sign arrow is present and whether each production caller of the re-signed function is in an Edit group."
---

Instance-level answers are in the plan's cards; each entry names a class of site to trace from the changed symbol outward.
