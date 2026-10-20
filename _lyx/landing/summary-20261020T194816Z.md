# Edit nothing until every lens fork has returned

A session that spawns lens forks could start acting on the first report while the others were still running.
Doing so loses the whole view: duplicates across lenses cannot be merged once the first is fixed, a later finding may cite text that has since changed, and of two findings proposing different changes at one spot the first applied tends to stay.
The wait itself costs little, since the forks run in parallel.

## Changes

- `contracts/stencils/twill/twill-directive.md`: the wait step now says the writer edits nothing while any fork is outstanding, whatever an early report says, and judges all findings together once the last fork has returned, before any edit.
- `contracts/stencils/burler/burler-fix-self-check.md`: the fixer's self-check wait step carries the same rule.
- `contracts/stencils/burler/burler-cluster-fan.md`: the fanned review handler keeps its holistic review going while forks run, but writes nothing to the review file until the last fork has returned, and consolidates with every finding in view.
- `internal/twilldirective/doc.go` and `internal/burlerengine/doc.go` describe the rule; the cluster-round paragraph of the burlerengine doc is reflowed to semantic line breaks.

## Design decisions

- The rule sits in each stencil's wait step, so the twill judge parts stay unchanged and host-neutral.
- The relaunch and resume rules, where a session has lost its forks, are unchanged: a lost fork never returns, and those rules already record it as "no report".
- Webster's implementer forks are out of scope: they run one per batch and are not review forks whose findings are judged together.
- The research documents under `docs/research/` that quote the old wording record past bench runs and are left as they are.

## Testing

The rule is prompt text with no mechanical check.
The `twilldirective`, `burlerengine` and `contracts/...` packages pass under `lyx gate test`.
