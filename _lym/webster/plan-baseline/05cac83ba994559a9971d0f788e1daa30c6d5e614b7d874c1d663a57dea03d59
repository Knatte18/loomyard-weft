# Card 4 — gate-config-reach

**Create:**
- `plan:internal/gateslot#ReportReach` -> `const ReportReach = "reach"`

**Edit:**
- `internal/gateslot#Config`
- `internal/gateslot#configFile`
- `internal/gateslot#decodeConfig`
- `internal/gateslot#decodeGates`
- `internal/gateslot#ConfigSpec`
- `internal/gateslot#GateKey`
- `internal/gateslot/template.yaml`
- `internal/gateslot/doc.go`
- `internal/gateslot#templateDefaults`
- `internal/gateslot#TestLoadConfig`
- `internal/loomcli#gateConfigWithTwoTiers`

**Intent:**
Accept the `reach` report and the new keys in the hub's `gate.yaml` (D12).

- `ReportReach` names the `reach` report step; its doc comment says what card 5's step does.
  `decodeGates` accepts `reach` beside `redundancy`; its refusal of an unknown report now names both: `report %q; want reach or redundancy`.
  `mutation` stays an unknown report name: its acceptance comes with D10's later build.
- `Config` and `configFile` gain `ReachBudgetMin` (`reach_budget_min`), `MutationMaxMutants` (`mutation_max_mutants`) and `MutationBudgetMin` (`mutation_budget_min`).
  `decodeConfig` refuses each below 1 with the same line its redundancy bounds use.
  `ConfigSpec`'s categories mark the three cross-pair, as the redundancy keys are.
- `template.yaml` gains the three keys after `redundancy_budget_min`, each with its comment:
  `reach_budget_min: 10` (minutes the reach step and `lym gate mutate` may spend refreshing the worktree's maps once they hold a slot),
  `mutation_max_mutants: 40` (the most mutants `lym gate mutate` runs for one diff, spread over the changed functions in turn),
  `mutation_budget_min: 20` (minutes `lym gate mutate` may spend on its mutant builds and runs).
  The `gates` comment's `reports` sentence names `reach` too, and says it runs before every other report step.
  The template lists `reach` first in the `reports` of every key that lists `redundancy` (`webster`, `parent-align`, `darn-write`, `code-burler-fix`), and gives `batch` `reports: [reach]`.
- `GateKey`'s `Reports` comment names both report names, and `doc.go`'s `reports` paragraph names the keys the template lists `reach` on, `batch` among them.
  `doc.go`'s list of load refusals names `reach` beside `redundancy` as an accepted report.
  Its bounds paragraph gains the three new keys and what each bounds, and says LoadConfig refuses each below 1.
  An existing hub picks the defaults up with no reconcile: config load fills a template key a tuned `gates` entry lacks, as the doc already states, and a scalar key the hub file lacks takes the template's value.
- `gateConfigWithTwoTiers`, the loomcli test helper that string-replaces the template's exact `code-burler-fix` line, changes its matched and replacement lines to `reports: [reach, redundancy]`,
  so the helper still declares `tiers: [0, 1]` on that key and its doc comment stays true for every test that calls it.

Tests (tier0):
- `templateDefaults` carries the three new values, `reach` first in the four keys' `Reports` and `batch`'s `Reports: []string{"reach"}`, so `TestTemplateConfigAndLimits` checks the template.
- `TestLoadConfig` gains rows: a key listing `reach` loads; each new key at 0 is refused naming it;
  a tuned `darn-write` without `reports` takes the template's `[reach, redundancy]`, and a hub `batch` without `reports` gets `[reach]` (the D12 criterion);
  the "an unknown report" row's wanted refusal becomes the new message;
  the rows "a tuned webster without reports takes the template's" and "a key's lints load in order", whose tuned `webster` and `parent-align` take the template's `reports`, now expect `Reports: []string{"reach", "redundancy"}`.

**ImpactSummary:** gateslot's config gains one report name and three bounded keys with template defaults; the only exact-string assertion of the old refusal is the `TestLoadConfig` row this card updates, the one test helper matching the template's `code-burler-fix` line follows it, and no reader of the keys exists until cards 5 and 13.
