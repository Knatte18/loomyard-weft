# Shipped stencils name no LoomYard PATTERN entry or lyx CLI rule

Every hub gets the shipped stencils and the deployed loom specs, but several of them cited LoomYard's own PATTERN entries and a `lyx` CLI-shape rule.
A reviewer in a hub without those files went looking for them, as the Plan-Burler-Review reviewer did in the quarry hub (issue #795).

## What changes

- The plan-review rubric's CLI-shape bullet, which checked a card adding a `lyx` command against `pattern/PATTERN-cli-cobra.md`, becomes a PATTERN-fit bullet.
  When the repository has a `PATTERN.md`, each card is checked against its entries and the `pattern/` background files the plan names as relevant;
  in a repository without one the check does not apply.
- The test-economy bullets of the plan, darn, webster and merge review rubrics keep their findings and drop the `PATTERN-test-economy` citation.
- The plan writer's card rule, `loom-plan-spec.md`'s matching rule and burler's fix step drop the same citation.
- The shed driver guide's stranded-branch repair drops its `PATTERN-fabric-git` citation and states the raw `git` call is an allowed repair.

## Design decisions

- The test-economy rule stays in the shipped rubrics: a redundant or missing test is a finding in any repository.
  LoomYard's stricter parts already live in `pattern/PATTERN-test-economy.md`, and its CLI rule in `pattern/PATTERN-cli-cobra.md`;
  the PATTERN directive renders LoomYard's `PATTERN.md` into its reviewers' prompts, so LoomYard's reviews keep both rules.
- Banner comments are left as they are, since `stencil.StripLeadingComment` removes them before a stencil reaches an agent.
- No guard test or PATTERN entry is added: `loom-plan-spec.md` documents the `fabric-vocabulary-name` gate, whose Go finding names `PATTERN-fabric-vocabulary` in every hub, so a scan would need an exception until that gate is made repo-neutral.

## Testing

No test pins the changed wording, and stencil prose is guarded by review.
`lyx gate test ./contracts/...` passes, covering the stencil marker and registry tests.
