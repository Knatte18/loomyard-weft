---
round: 1
focus:
  - "internal/yamlengine/set.go: does a dotted path into a nested map or struct field write the scalar at the right depth, so `tiers.tier0.env.LYM_TEST_TIER=0` no longer lands one level too high or corrupts the config?"
  - "internal/yamlengine/set.go: is a path into a struct field the schema lacks refused with an error that names the key, and are top-level keys and single open-map entries still set as before?"
  - "internal/yamlengine/set.go and internal/gateslot/config.go: how are empty or malformed path segments, a path that runs through a scalar or a list, and a key containing a dot handled; is the written value still validated against the schema, and is a failed set left without a partial write?"
  - "internal/configcli/configcli.go, internal/configengine/set.go, internal/yamlengine/layers.go: do the changed signatures and error paths keep every caller consistent, and does the refusal reach the operator unchanged?"
  - "Tests in internal/yamlengine/set_test.go, internal/yamlengine/layers_test.go and internal/gateslot/config_test.go: does each new behavior have coverage, and does any new test duplicate an existing one?"
  - "doc.go files and internal/gateslot/template.yaml: do the doc comments follow the repository's comment conventions, and do they match the new behavior of `--set`?"
---

The decision record holds only the board entry under `## Goal`, so the measuring stick is its Idea: `--set` takes a dotted path to any key the schema reaches, nested maps included, and refuses a path into a struct field the schema lacks, naming the key.
