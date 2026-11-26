---
round: 2
focus:
  - "Is every config key named in full, `<module>.<dotted path>`, in Go messages, flag help, comments, package docs and docs/? Search from the keys of every module's template rather than from a site list, including gatecli, batcher, battencli, beamcli, webstercli, hibernatecli, preflightshed, weaverengine, sheddriver, shedarbiter, burlerengine, loomshed, loomcli, hardencli, loomengine, driftengine, hardenshed, reportcli and docs/overview.md. Is each Go message rendered through configengine.KeyName from a named constant?"
  - "Do the reconcile help and package doc in internal/configcli, docs/overview.md, docs/shared-libs/configengine.md and internal/preflightshed/doc.go describe which reconciles move or drop an old key, and which verbs run the hub walk, as the code behaves?"
  - "Would a test fail if internal/hubreconcile/walk.go mailed the moved-keys notice only after a successful walk?"
  - "Would a test fail if the reconcile-before-load call were removed or reordered in internal/battencli/arm.go, internal/fabriccli/merge_verbs.go, internal/beamcli/cli.go, internal/webstercli/wiring.go or internal/shuttleparity/seats.go?"
  - "Does moving keys in internal/yamlengine/move.go keep the output parseable when one moved value is a YAML alias of another, and what happens in internal/configsync/configsync.go to a profile or recipe block whose name holds a dot?"
  - "Do the move tables, the area-named old key branch in internal/configengine/set.go, the null and already-set drop outcomes in internal/configreg/runsettings.go and the move error file paths in internal/configsync/configsync.go have tests with independently stated expected values?"
---

The entries cover the open areas of the whole range.
They set no severity and no verdict.
