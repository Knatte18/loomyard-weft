# Hardener rounds default to sonnet[high]; a rotation is set per run

The hardener template rotated `reviewer` and `fixer` through `opus`, `fable` and `sonnet` round by round.
That rotation came from the hardener's own run, not from an operator choice.
Fable is too dear for a default, and what a round finds rests on its seeded focus more than on its model.
The template now sets both lists to `["sonnet[high]"]`, one model with no rotation.
An operator who wants a rotation sets `hardener.reviewer` and `hardener.fixer` as lists in the task's `run_settings`.

## What changes

- `internal/hardenshed/template.yaml`: the `reviewer` and `fixer` defaults are `sonnet[high]` alone, and their comments say a rotation is a per-run choice.
- `internal/hardenshed/campaign.go`: `CheckCampaign` no longer refuses a one-entry reviewer list.
  The one-entry list is the new default, so the old refusal would have rejected every default campaign.
  Lists of two or more entries are still refused when the cyclic expansion repeats a model in neighbouring rounds.
- `internal/hardenshed/doc.go`: the package doc states the one-model default and drops the claim that rounds rotate models.
- `contracts/stencils/harden/harden-rubric-review.md`: the judge's CONVERGED rule no longer requires the reviewer model to differ from the previous round's.
  With one model by default, that clause could never be met, so no campaign could converge.
- `run_settings` already takes `hardener.reviewer` and `hardener.fixer` as lists; `TestPrepareRunSettings` in `internal/configreg` now pins that.

## Design decisions

- The refusal of equal neighbouring models stays for multi-entry lists, so a per-run rotation still cannot repeat a model back to back.
- The CONVERGED rule keeps its other two conditions: no BLOCKING or MEDIUM finding, and a green live gate.

## To check by hand

- The judge's rule for CONVERGED in `contracts/stencils/harden/harden-rubric-review.md` no longer reads the reviewer model.
  Check that no judge prompt or Go code outside that stencil still expects a model change as a condition.
  A grep of `internal` and `contracts` found none, but the finding was raised as BLOCKING and is worth a second look.

## Merge danger

**Door:** two-way
**Blast radius:** hardenshed
