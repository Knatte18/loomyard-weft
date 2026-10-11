# Card 5 — landing-deploy-key

**Edit:**
- `internal/landingshed#Config`
- `internal/landingshed/template.yaml`
- `internal/landingshed/doc.go`
- `internal/landingshed#TestLoadConfig_TemplateDefaults`
- `internal/configcli#TestConfigOverRealHub`
- `cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `plan:internal/landingshed#TestLoadConfig_Deploy` -> `func TestLoadConfig_Deploy(t *testing.T)`

**Intent:**
`landing.yaml` gains the hub-wide `deploy` key of D2, which card 9 has batten's Land row read.

`landingshed.Config` gains `Deploy string` (yaml `deploy`): the shell command batten's Land row runs in the prime after a landing into the prime's branch.
`template.yaml` gains `deploy: ""` with a comment: the command runs in the prime after a landing into a branch listed in `require_pr_to_base`; empty runs nothing; the same trust class as `publish_verify`.
The template default stays empty, so lyx stays repo-generic; the hub sets `./update-plugins.sh`, which is the orch's step during this task's own landing, not this card's.

`internal/landingshed/doc.go` gains a paragraph on `deploy`:
it is read only by batten's Land row, from the board dir like the rest of landing config, never from a pair;
it runs only for a parent branch in `require_pr_to_base` when the row runs, so it executes code PR-Gate reviewed, except that an edit to that list between PR-Gate and Land, which only the orch's own hub config edit can make, deploys a tip PR-Gate passed unreviewed;
and `lyx config` refuses a write to it from a pair, as for every hub-wide key, a guard against a mistaken write rather than a determined agent.
No guard is added: `configcli`'s existing hub-wide write refusal already covers every landing key.

Tests:
`TestLoadConfig_TemplateDefaults` asserts `cfg.Deploy` is empty;
the new `TestLoadConfig_Deploy` loads a landing file with `deploy` set and one without it, and asserts the set value and the empty one;
`TestConfigOverRealHub`'s hub-wide refusal table gains a row writing `landing` `deploy=./update-plugins.sh` from the task pair, refused naming the prime worktree.
The new top-level test raises `internal/landingshed`'s row in `cmd/lyx/testdata/test-budget.yaml`.

**ImpactSummary:** one additive landing key with an empty default; existing hubs gain it through hub-wide reconcile, and no landing producer reads it.
