---
round: 2
focus:
  - "Upgrade path of an existing hub: after the reconcile (internal/yamlengine/follow.go, internal/configsync/configsync.go, internal/hubreconcile/walk.go, internal/configcli/hubwide.go) of a profiles.yaml built from the pre-change internal/runprofile/template.yaml, does runprofile.Load accept every file that can be committed? Cover an operator-repointed defaults.arras, a default written through a YAML alias, edited and operator-added arras blocks, and a missing template base."
  - "Any step that rewrites a hub's profiles.yaml during that reconcile, with its registration in internal/configreg/configreg.go: does it run once, on every reconcile caller, change only what makes the file unloadable, keep the operator's other content, and report what it dropped or left? Does a test drive the upgrade path through to a successful load?"
  - "The change description at _lym/landing/summary.md: do its Merge danger section (door and blast radius, including each existing hub's committed profiles.yaml and profiles.template-base) and its What changes bullets on the reconcile and on a kept defaults.arras match what the committed diff does?"
---
