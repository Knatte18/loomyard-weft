---
round: 1
focus:
  - "internal/yamlengine/follow.go: does FollowTemplate's three-way merge settle each case in the record's rule 2 correctly (hub equals base and template moved, hub differs from base, template added a key, template removed a key), including nested maps, lists and a key present only in the base?"
  - "internal/hubreconcile/walk.go and internal/configsync/configsync.go: the base file kept beside each seed-only file. Is it written, committed on Bolt with the file, and read back correctly, and does a failure to read or write it leave the hub file untouched?"
  - "No-base path (record rule 3): with no base yet, does the code take the template as seeded where that can be told, and otherwise add a lacking key and keep and report a differing value? Does the loomyard hub outcome match the record: `code_reviewer_fan: \"\"` added to `docs` and `speed`, `light` left with the fan?"
  - "Seed-only coverage (record rule 4): does every module the hub reconcile treats as seed-only follow the template, and does the change description list them accurately against the diff?"
  - "Reporting (record rules 2 and 5): `updated` and `differs` entries in the `lym config reconcile --layer default` dry run, and the orch mail after a walk. Are both values given for a differing key, and does the dry run write nothing?"
  - "internal/runprofile/template.yaml: is `loom.code_reviewer_fan: \"\"` removed from every recipe block of `light` and kept in `docs` and `speed`? Check the migrate.go change against the removed Lacking hook and the former-name pairs it now lists."
  - "Docs the record names: internal/runprofile/doc.go and the reconcile docs (hubreconcile, configsync, configcli, configengine doc.go, docs/overview.md, PATTERN.md and pattern/PATTERN-config-layers.md) must state the new rule and no longer say a present file is never filled or pruned. Check for leftover old wording."
  - "Test economy: the changed tests in configsync_test.go, follow_test.go, configcli_integration_test.go and ensure_integration_test.go. Look for duplicated coverage between the yamlengine unit tests and the integration tests, and for added behavior with no test."
---

Seed for round 1, set from the decision record's `## Goal` and the commit range's file list.
