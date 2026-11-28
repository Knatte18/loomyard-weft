---
round: 1
focus:
  - "Upgrade path of an existing profiles.yaml: when the hub reconcile (internal/configsync) drops the arras keys key by key and the operator has changed some of them but not others (for instance defaults.arras repointed to another profile whose arras block is untouched, or an operator-added default or profile), does the resulting file still pass runprofile's load, including decodeDefaults' rule that a default names a profile holding that recipe's block? Settle what the reconcile and its load check do in that case, and what it means for runs of other recipes."
  - "Consumers of a profile's arras block or the arras default (batten Seed-Child's prepareRunSettings, loom's run usage record, board profile and run_settings handling, raddle context loading): is the behavior with no arras block or default what the board entry asks, both for an arras run naming no profile and for one naming a starting profile?"
  - "The change description's Merge danger section: check the two-way door and the runprofile blast radius against what the template change does to an existing hub's profiles.yaml through the reconcile."
  - "The change description's claims about the reconcile and about the configsync key-move test carrying the arras keys' drop: read internal/configsync and its tests and settle whether each claim holds, and whether removing keys from an existing file has coverage."
  - "The board entry's docs ask: read docs/overview.md and docs/reference/weaving-glossary.md as the diff leaves them, and settle whether every recipe description naming arras marks it as kept in code but not offered, and whether the edited sentences stay accurate."
---

The diff is small, and its behavior change reaches an existing hub through the reconcile rather than through new code.
The upgrade path is therefore the first place to look.
