---
verdict: APPROVED
---
The range `0728a1475..HEAD` holds one commit, `56763d2e9`, which changes the wording of three stencils and no Go code.
No lens and no part of the handler's holistic review found a defect.

Each focus entry, settled:

- **Coverage of the entry's `Expected`.**
  The darn writer's two Step 4 fragments and the sampler agent's commit line now tell those agents to commit after each step, and each gives the entry's reason.
  The other agents that edit and commit over several steps already commit per unit of work:
  - the weaver batch and the webster implementer commit once per card (`weaver-template-batch.md:66`, `webster-body-implementer.md:78`);
  - the landing fixer, the darn gate fixer and both verify fixers commit once per fix (`landing-template-fixer.md:45`, `gatefix-body-darn.md:23`, `weaver-body-verify-fix.md:38`, `webster-body-verify-fix.md:35`);
  - the burler source fixer commits each finding or group once it is green (`internal/burlerengine/prompt.go#fixScopeRules`).

  The Discussion writer, raddle, twill fork0 and the card cold fixer commit nothing.
  No shipped skill under `plugins/ly/skills` has an agent edit and commit over several steps.
- **The change description's `Design decisions`.**
  Each claim holds against the stencil and skill files.
  The list of agents that already commit per fix leaves out the burler source fixer, but that fixer also commits per fix, so the conclusion is unchanged.
- **The darn writer's two fragments.**
  Read inside `darn-template-write.md`, the new step 6 agrees with the steps around it.
  It agrees with step 1's "Each commit is one change", with step 2's keeping the commits in the writer's own session, and with step 3's gate after each commit's last edit in the ungated fragment.
  The gated fragment's writer runs no gate, so it commits once a step's edits are made.
  Docs and PATTERN edits still go in the same commits as the code.
  The Step 3 stop paths, which end with no commit, come before any Step 4 edit.
  Step 6 is the last item in the list, but its own text says to commit after each step as soon as the step is made, and step 1 already sets one change per commit.
  A writer reading the list in order therefore commits while working.
- **The sampler's commit line.**
  It agrees with the trial section's "Commit what the trial settled as an ordinary change" and with the ending's clean-worktree commit.

The commit message matches the diff.
No declared PATTERN changes exist, no trailer is needed, and the summary's two-way door and stencil blast radius fit a wording-only change to stencils read at call time.

## Rejected

- The new wording calls the unit of commit a "step", the same word the darn template uses for its own headings, so a reader might mistake the two. Rejected: step 1 sets the unit as "one change", and step 6's "as soon as its edits are made / its gate passes" can only mean a unit of work inside Step 4.
