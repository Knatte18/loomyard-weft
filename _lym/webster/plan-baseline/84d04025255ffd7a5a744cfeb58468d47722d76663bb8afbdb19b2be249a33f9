# Card 11 — loosened-check-fragment

**Create:**
- `contracts/stencils/loom/loom-fragment-loosened-check.md`

**Edit:**
- `contracts/stencils#entries`
- `contracts/stencils#TestStencils_TopLevelMarkers`
- `contracts/stencils/webster/webster-body-implementer.md`
- `contracts/stencils/webster/webster-body-verify-fix.md`
- `contracts/stencils/weaver/weaver-template-batch.md`
- `contracts/stencils/weaver/weaver-body-verify-fix.md`
- `contracts/stencils/darn/darn-template-write.md`
- `contracts/stencils/sampler/sampler-template-agent.md`
- `contracts/stencils/burler/burler-step-fix.md`
- `contracts/stencils/gatefix/gatefix-body-darn.md`
- `contracts/stencils/landing/landing-template-fixer.md`
- `contracts/stencils/landing/landing-template-pr-review.md`

**Uses:**
- `internal/stencilstore#ReadForEngine`

**Intent:** D2: one shared fragment states the `Loosened-Check:` trailer, and every brief of a session that commits code includes it (`PATTERN-stencil-ownership`).

`loom-fragment-loosened-check.md`, with a leading banner naming its includers, says, worded for any engine and language:
a commit that widens or removes an assertion, tolerance or limit in a test that exists before the change records each one as a trailer line in the commit message's last paragraph, `Loosened-Check: <test> — <what was loosened> — <measured cause>`, one per check;
the cause is a measurement, never "flaky" or "to make it pass";
a session that forgot one adds it in a later commit, with no change if need be, never by rewriting history;
Code-Review and Drift read it.
It carries no marker and no engine block.

It is registered with its embedded byte var in `stencils.go`'s `entries`, and `TestStencils_TopLevelMarkers` gains its row with no marker.

Each brief listed includes it with the bare `{{template "loom-fragment-loosened-check"}}` beside its commit rules: the Webster and Weaver implementers and verify fixers, the recovery strand through `webster-body-implementer`, the darn writer, the sampler agent, the burler fixer that Code-Burler-Fix runs, the darn gate fixer, the landing merge fixer and the PR-Review agent.
Each brief is read through `ReadSet` or `ReadForEngine`, which resolve the include after card 6.
`gatefix-body-card`, `landing-template-conflict`, `landing-template-fix-review` and `webster-template-master` commit no code, so they do not include it.

**ImpactSummary:** One new registered fragment included by the ten code-committing briefs; no marker or Go value changes.

**Commit:** `11: loosened-check-fragment`
