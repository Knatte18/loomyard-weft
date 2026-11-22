---
round: 3
focus:
  - "internal/hardenshed/campaign_rows.go and internal/shedadapters/round.go: does the charter path `Campaign.RoundFacts` names come from the one focus-path builder the Bouncer's seed, judge outputs, focus reader and resume probe use, with no second declaration of the `round-%d-focus.md` spelling in production code, and does every caller of the renamed builder still compile and behave the same?"
  - "internal/configreg/configreg.go, internal/gateslot (template.yaml and the new rewrite): for a hub gate.yaml that still holds `gates.harden-live-copies`, does a `configreg.Reinterpretation` on the gate module either remove the key when `harden-live` names exactly one tier and it equals the copies tier, or leave `harden-live` unchanged and return a flagged line naming the way forward; and does a test reach the rewrite for the removal case, the flag text, and a file without the key left unchanged?"
  - "_lym/landing/summary.md against the diff: do its Design decisions, its hub-gate-writes bullet and its Merge danger door and blast radius state that the retired copies key is folded into `harden-live` and that a gate.yaml meaning changes?"
---

Targets the code the round 2 fixes touch, since those edits have not been reviewed.
