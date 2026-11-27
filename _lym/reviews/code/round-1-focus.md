---
round: 1
focus:
  - "internal/gatecli/run.go, tiers.go and scope.go: with markedScope's unlisted-tier refusal gone, trace a `--gate upkeep` call whose marked tests sit in a tier the key does not list. Does every later step treat that tier correctly, including its command, slots, admission, tier settings, flags such as --tier, green records and run records, or does any step still rely on the listed tiers covering it?"
  - "internal/gatecli guard step: is a marked test of a tier the hub does not declare still refused, as only.go, doc.go and template.yaml now describe, on every flag combination, with a refusal whose way forward fits a marked key? Does any test cover that behaviour for the marked scope?"
  - "`_lym/landing/summary.md`: check each entry under Changes and Design decisions against `git diff --stat` and the diff of the review range, including the list of files said to state the new rule (compare internal/gatecli/cli.go). Check its Merge danger door and blast radius too."
  - "internal/gatecli (doc.go, cli.go help and comments, only.go, scope.go), internal/gateslot (doc.go, tiers.go, template.yaml), repository docs and pattern/: does any text still state or imply the removed rule that a marked key's tiers must cover its marked tests, or contradict the new text about what the key's tiers do for scope `marked`?"
  - "internal/gatecli/scopes_integration_test.go: does the rewritten case prove that the tier1 marked test runs under tier1's own command and settings, and does the surrounding test's name and doc comment still match what it checks?"
---

Seed focus for round 1.
The review range holds the commits over `bae1e3af7f592d5bfc7b25b5e36a736e24f76a7f`, and the answer key is the decision record alone.
The board entry's task allows either a changed template default or deriving the tiers from the marked tests.
Setting the hub's key and the hand check on main are left to the orch by that entry.
