---
round: 1
focus:
  - "internal/batcher/template.yaml: does `wide` equal `cautious` with `budget: 300000` and nothing else, does `active:` name it, and do `cautious` and `identity` remain with unchanged weights?"
  - "internal/configsync/configsync.go: the former-default strip of `active: cautious` on reconcile. Is it applied only to a value that equals the old template default, so a hub that chose `cautious` deliberately is not silently moved, or is that unavoidable and acknowledged? Does it handle absent and non-string `active:` values?"
  - "internal/batcher/config.go and registry.go: do the changed lines still resolve the default profile consistently with the new template, with no stale `cautious` fallback left?"
  - "Comments and docs (docs/overview.md, internal/batcher/doc.go, internal/websterengine/doc.go, template.yaml): is every place that names `cautious` as the template default updated, with none left stale (grep for it)?"
  - "Tests (config_test.go, configsync_test.go, verbs_test.go): do they cover the new default and the strip behavior without duplicating existing coverage, and does any rewritten test lose a case it covered before?"
---

Seed focus from the decision record's Goal and the diff stat.
