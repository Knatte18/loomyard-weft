```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens. I mapped each change in the diff to the board entry's goal points, and each goal point back to the changes that build it.

**Decisions.** The answer key is a decision record alone. It has no `Added after Discussion` entries, and the board entry's `## Expected` holds two goal points:
- **G1:** agents that work iteratively, darn writers above all, commit after each coherent step instead of holding the whole change uncommitted until the end.
- **G2:** the writer stencils and skills say so, with the reason: a stall, a dead strand or a reboot loses at most one step, and the parent sees progress in the branch's history.

**Per change:**
- **C1** — `contracts/stencils/darn/darn-fragment-make-change.md` step 6, for the darn writer without a cold fixer: commit after each coherent step once its gate passes, never hold the change uncommitted, with the reason. It traces to G1 and G2.
- **C2** — `contracts/stencils/darn/darn-fragment-make-change-gated.md` step 6, for the darn writer with a cold fixer: the same rule, committing once a step's edits are made, with the same reason. It traces to G1 and G2.
- **C3** — `contracts/stencils/sampler/sampler-template-agent.md:34-35`: the sampler agent's commit-as-you-go line now forbids holding the work until the end and gives the reason. It traces to G1 and G2. The sampler is an iterative agent that commits over several steps.

**Per decision:**
- **G1** is built by C1, C2 and C3. The other iterative agents that write code already commit per unit of work:
  - weaver batch, one commit per card: `contracts/stencils/weaver/weaver-template-batch.md:66`
  - webster implementer, one commit per card: `contracts/stencils/webster/webster-body-implementer.md:78-79`
  - landing fixer, one commit per fix: `contracts/stencils/landing/landing-template-fixer.md:45`
  - darn gate fixer: `contracts/stencils/gatefix/gatefix-body-darn.md:23`
  - verify fixers: `contracts/stencils/weaver/weaver-body-verify-fix.md:38` and `contracts/stencils/webster/webster-body-verify-fix.md:35`
  - hardener fix, commit with the fix: `contracts/recipes/hardener-recipe.yaml:152`

  The remaining agents commit nothing:
  - Discussion writer: `contracts/stencils/loom/loom-template-discussion.md:171`
  - raddle agent: `contracts/stencils/raddle/raddle-template-agent.md:87`
  - twill fork0: `contracts/stencils/twill/twill-writer-fork.md:20`
  - card cold fixer: `contracts/stencils/gatefix/gatefix-body-card.md:24`

  No uncovered iterative writer remains.
- **G2** is built by C1, C2 and C3 for the stencils. No skill change is needed: none of the shipped skills under `plugins/ly/skills` has an agent edit and commit over several steps. The only commit mention in a skill is `plugins/ly/skills/board-policy/SKILL.md:61`, which says an exploration's report lands as a commit and instructs no writer. `contracts/specs/implementer-spec.md:37` only states that an implementer's output is commits; the per-card stencils carry the instruction.

**Findings:** none. Every change traces to G1 and G2, both goal points are built, and no commit carries a `Pattern-Departure:` trailer, nor needs one.

Tool calls:
1. `plugins/ly/skills/board-policy/SKILL.md:50-70`, and a commit grep of the distill, refresh and cli skills. My context held only the grep hit at line 61, so I could not judge whether a skill instructs an iterative writer.
2. A commit grep of `contracts/` outside `stencils/`. The manifest covered stencils only, and specs or recipes could also instruct writers to commit.
3. Commit lines in `contracts/specs/implementer-spec.md` and the hardener, arras and pawl recipes, to settle whether a spec or recipe instructs an iterative writer the diff left alone.
