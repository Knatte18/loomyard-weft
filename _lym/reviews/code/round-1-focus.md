---
round: 1
focus:
  - "`internal/orchengine/template.yaml` and `template.go`: do the defaults read exactly `model: opus` and `effort: medium`, and does `effort: medium` resolve in shuttle's engine vocabulary for the orch's engine?"
  - "Every path that starts an orch session, `lym s` and `lym beam start` included (see `internal/orchcli/start.go`): does each read `orch.model` and `orch.effort` from the loaded config, with no path still passing an empty `ModelChoice` or a hard-coded value?"
  - "Does anything else rely on the old empty default, such as docs, other templates, config-layer merge code, golden files or tests that assert the template text, and does the diff land the docs the repository's rules name for a config default change?"
  - "The template comments now say an explicit empty value uses the provider default: does the loader and `start.go` still honor an explicit empty override as the comment claims?"
  - "`TestLoadConfig_TemplateResolvesWithNoFile` in `internal/orchengine/config_test.go`: is the changed assertion the only coverage the new default needs, or is a start-path behavior left without a test?"
---

The diff is one commit touching the template, its doc comment and one test.
The decision record's `## Goal` carries the whole answer key; its other sections are empty.
The commit message claims every start path builds its spec from the loaded orch config, so that claim needs a check against the code.
