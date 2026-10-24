---
round: 2
focus:
  - "internal/configsync/movekeys.go takeKey and internal/configsync/configsync.go formerTemplateValues: settle whether a darn.yaml value equal to darn's former template default for a moved key (review_hard_max_bounces 2, darn_reviewer_model sonnet[high], empty darn_fixer_model and fan keys) is dropped rather than moved over a loom.yaml value, and whether a genuinely tuned darn value still moves."
  - "internal/configreg/keymove.go and ReconcileHubWideAt: settle whether the moved-key set stays exempt from the strip pass correctly once a dropped value is reported, so no key is left unmoved and unreported."
---

The focus covers the commit range's fix to the key-move rule; the remaining findings are mechanical.
