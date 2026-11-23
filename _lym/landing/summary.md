# Hardener rounds default to sonnet[high]; a model rotation is set per run

The hardener template's `reviewer` and `fixer` defaults rotated every campaign through `opus`, `fable` and `sonnet`, round by round.
The operator never chose that rotation.
Fable costs too much for a default,
and what a round finds depends on its seeded focus rather than on its model.

## What changes

- `internal/hardenshed/template.yaml` sets `reviewer` and `fixer` to `["sonnet[high]"]` alone.
  Each key's comment says the default is one model with no rotation.
  It also says a rotation over several models is the operator's choice per run, set in the task's `run_settings` as `hardener.reviewer` or `hardener.fixer`.
- The `hardenshed` package doc's Config section states the same default and per-run choice.
  Its Method section no longer describes rotation as how every campaign runs.
  It names rotation as an option the operator sets for a run that wants agreement across models.
- `TestLoadConfig`'s template defaults follow the new values.

## Preflight accepts a one-entry reviewer list

`CheckCampaign` refused a reviewer list of one entry, which would have rejected the new default.
It now passes a one-entry list as a single-model campaign.
It still refuses a list of two or more entries whose cyclic expansion puts one model in neighbouring rounds.
`TestCheckCampaign`'s `ReviewerListOfOneEntryPasses` row pins the single-entry case.

## run_settings already takes lists

Both keys are open to a task's `run_settings`, and `PrepareRunSettings` passes list values through to the local `hardener` document.
`CheckValues` then parses each entry as a model-spec.
No code needed a fix.
A new `TestPrepareRunSettings` row checks this: a `hardener` object holding `reviewer` and `fixer` lists produces a document carrying both lists in order.

The hub's own `hardener.yaml` sets neither key, so the new template default takes effect there once this lands.
A hub file that wrote the old rotation out explicitly keeps it, which is that hub's own setting.

## Merge danger

**Door:** two-way
**Blast radius:** hardener
