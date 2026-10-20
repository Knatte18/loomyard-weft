# Decision record — fork-findings-wait-all

## Goal

A session that spawns lens forks and acts on their findings edits nothing until every fork has returned, then judges all findings together, then edits.

## Scope

- `contracts/stencils/twill/twill-directive.md`: step 3 states the hard rule.
- `contracts/stencils/burler/burler-fix-self-check.md`: step 3 states the same rule.
- `contracts/stencils/burler/burler-cluster-fan.md`: the handler writes no part of the review file until every fork has returned, then consolidates.
- The package docs that describe these procedures: `internal/twilldirective/doc.go` and `internal/burlerengine/doc.go`.
- Out of scope: Webster's implementer forks, which run one per batch and are not review forks, and the research documents under `docs/research/`, which record past runs.

## Decisions

### The rule lives in the wait step

Decision: each stencil's wait step says the session edits nothing while any fork is outstanding, and that the judging step starts only once the last fork has returned.
Rationale: the wait step is where a writer decides what to do with an early report, and the judge parts stay host-neutral and unchanged, since they already judge every finding.

### The cluster handler holds the review file

Decision: the fanned cluster handler keeps its holistic review going while forks run but writes nothing to the review file until every fork has returned.
Rationale: the handler acts on fork findings by consolidating them into the review file, so the same loss of the whole view applies to a review written early; its holistic reading during the wait stays allowed.

### Lost forks keep their existing rule

Decision: the relaunch and resume rules, where a session has lost its forks and judges the findings it holds, stay as they are.
Rationale: a lost fork never returns, so waiting for it would never end; those rules already name it as "no report".

## Constraints

- Stencils are host-neutral: the twill directive and its parts name no file, step, verb or section of any one host.
- Semantic line breaks in every changed `.md` and Go comment.
- `PATTERN-stencil-ownership`: prompt text stays in stencils read at call time.

## Auto-mode assumptions

- "Edit" covers every write the session makes to the files it reviews or fixes, and for the cluster handler the review file.
- Reading code while forks run stays allowed, since it changes nothing a finding cites.

## Open risks

- The rule is prompt text only; no gate checks that a session edited nothing while a fork was outstanding.

## Acceptance criteria

- The rendered twill directive states that nothing is edited until every fork has returned, and that all findings are judged together before any edit.
- The rendered burler fix self-check states the same.
- The rendered fanned cluster rules state that the review file is written only after every fork has returned.
- Existing twilldirective and burlerengine tests pass.
