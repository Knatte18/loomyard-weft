---
round: 2
focus:
  - "Check that `internal/loomengine/template.yaml` ends with a newline and that the `TestLoadConfig_Loads` rows built as `ConfigTemplate() + \"<key>: <value>\\n\"` (`selfreport: false` and the retired `fix_start` row) put their key on its own line, so each reaches the YAML parser and exercises the loader."
  - "Check that `internal/shuttleengine/doc.go` names the hold deferral constants without restating values that `hold.go` declares."
---

Both entries target text changed after round 1's review, which no fresh reviewer has seen.
