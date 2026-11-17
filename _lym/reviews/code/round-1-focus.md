---
round: 1
focus:
  - "internal/fabricengine/config.go ConfigSpec: with branch_prefix as CategoryMachine on a HubWide spec, settle whether the machine layer is actually read by LoadConfig and by every other loader of fabric's config, and whether the hub-wide refusal path still rejects a local-layer branch_prefix with a sensible message."
  - "Scope against the board entry: it asks for a per-machine or per-user layer for branch_prefix and a review of the other personal keys (PR identity, Slack operator, default model or effort, batten.open_ide). Settle whether the diff covers the review half or leaves it undone, and whether the change description or commits say so."
  - "internal/configcli/configcli_test.go TestDispatchLayers: the case was repointed from fabric to slack. Settle whether slack is a hub-wide module with no machine key, so the refusal and its wantErr still hold, and whether any fabric machine-layer path (set, get, unset, refusal of other fabric keys) is left without coverage."
  - "Settle whether the pattern/PATTERN-config-layers.md edit and the new Machine-category wording in configengine/layers.go and doc.go match each other, and whether other docs or specs that list machine keys or say fabric has no machine key (docs, specs, stencils, error text, grep for 'cross-pair' and 'no machine key') still contradict the change."
  - "internal/fabricengine/config_test.go: the new subtest inside TestLoadConfig; check it for test economy against TestDispatchLayers and TestModules_EveryTemplateKeyClassified, and settle whether the claimed 'local pathspec or warp exclude list is refused' coverage in the doc comment is pre-existing or new."
  - "Settle whether any code derives a branch name or compares a branch against the prefix in a way that assumes one hub-wide value, now that two machines may use different prefixes (fabricengine slug-from-branch, teardown, adopt, beam)."
---

Seed focus for round 1. The decision record holds only the imported board entry under `## Goal`; its other sections are empty.
