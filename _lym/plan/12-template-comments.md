# Card 12 — template-comments

**Prosa:**
- `internal/agentconfig/template.yaml`
- `internal/batcher/template.yaml`
- `internal/battenshed/template.yaml`
- `internal/boardengine/template.yaml`
- `internal/creelconfig/template.yaml`
- `internal/darnengine/template.yaml`
- `internal/desktophook/template.yaml`
- `internal/dobbyengine/template.yaml`
- `internal/driftengine/template.yaml`
- `internal/envconfig/template.yaml`
- `internal/fabricengine/template.yaml`
- `internal/gateslot/template.yaml`
- `internal/hardenshed/template.yaml`
- `internal/heddleengine/template.yaml`
- `internal/hibernateconfig/template.yaml`
- `internal/ideconfig/template.yaml`
- `internal/landingshed/template.yaml`
- `internal/languageconfig/template.yaml`
- `internal/lenses/template.yaml`
- `internal/loggerconfig/template.yaml`
- `internal/modelspec/template.yaml`
- `internal/orchengine/template.yaml`
- `internal/reedengine/template_posix.yaml`
- `internal/reedengine/template_windows.yaml`
- `internal/reportconfig/template.yaml`
- `internal/samplerengine/template.yaml`
- `internal/sheddriver/template.yaml`
- `internal/shuttleengine/template.yaml`
- `internal/shuttleengine/claudeengine/template.yaml`
- `internal/shuttleengine/codexengine/template.yaml`
- `internal/slackengine/template.yaml`
- `internal/weaverengine/template.yaml`
- `internal/websterengine/template.yaml`

**Intent:** Every other module's template comment names a config key by its full dotted name (D3), such as `beam.pull_request`, `fabric.branch_prefix`, `drift.judge_ask_parent` and `loom.code.max_bounces`, a key of the template's own module included.
No key, value or key order changes (D2); only comment text does, rewritten as prose where a comment cross-references a key.
A command a comment prints keeps the CLI's form, `lym config beam --set pull_request=merge`.
The loom, tapestry and profiles templates were rewritten by card 4.
