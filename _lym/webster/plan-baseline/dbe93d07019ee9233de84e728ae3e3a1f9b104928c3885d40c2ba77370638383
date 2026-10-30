# Card 2 — beam-pull-request-key

**Edit:**
- `internal/dobbyengine#Config`
- `internal/dobbyengine#ConfigSpec`
- `internal/dobbyengine#LoadConfig`
- `internal/dobbyengine#TestLoadConfig`
- `internal/dobbyengine/template.yaml`
- `internal/configreg#TestModules_EveryTemplateKeyClassified`

**Extends:**
- `internal/dobbyengine/doc.go`

**Intent:**
D1: `beam.yaml` gains `pull_request`, the receiving branch's pull-request mode.
`template.yaml` gains `pull_request: on  # whether a landing into this worktree's branch goes through a GitHub pull request: on (unless the entry's pull_request field is off), off (only when the entry's field is on) or required (always, whatever the field says)`, quoted as the YAML string `"on"` so it never parses as a bool.
`Config` gains `PullRequest string` with tag `yaml:"pull_request"`; the three values are plain strings, which card 8's resolver compares.
`ConfigSpec` classifies `pull_request` as `configengine.CategoryCrossPair`, since it decides what lands on the parent, so the local layer refuses it.
`LoadConfig` refuses any value but the three, after the negative-number checks, with `configengine.MarkInvalid` and the message `beam config key "pull_request" is "<value>": must be on, off or required; way forward: fix it with lym config beam --set pull_request=<on|off|required>`.
`TestLoadConfig` gains rows through its existing worktree-layer writer (`writeBeamLayer`): no layer setting it resolves `on`; `required` and `off` in the worktree layer resolve as set; `maybe` is refused naming the key and the way forward.
The local-layer refusal is pinned by `TestModules_EveryTemplateKeyClassified`'s `wantLocalDenied` row below and configengine's own local-layer tests, so `TestLoadConfig` adds no row for it.
`TestModules_EveryTemplateKeyClassified`'s `wantLocalDenied` gains `"beam": {"pull_request"}`.
The package doc's beam.yaml paragraph gains one sentence naming the key, its three values and that it is read with the receiving branch's worktree as the anchor.

**ImpactSummary:** Adds one cross-pair key to beam.yaml; dobby's existing readers ignore it, and only the configreg classification test pins the new local denial.

**Commit:** 2: beam.yaml: pull_request mode on, off or required
