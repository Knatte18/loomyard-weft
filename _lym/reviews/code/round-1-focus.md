---
round: 1
focus:
  - "internal/operatoragent/spec.go and its stencil: render the whole opening prompt for a task worktree and for the prime. Does parentdirective.Directive (its third argument, and its no-parent variant) agree with an agent the operator talks to in its own pane, and do the edit and PATTERN directives agree with the stencil's Start and Limits sections?"
  - "Same instructions, guards and skills as any lym session of the engine: trace Runner.StartSession for this spec on both engines. Do the agent role's agents.yaml list, the language skills, the shell guards and the standing passages all reach the session, and does anything keyed by role or seat (seat catalog, manifest, model or effort defaults, opening-stencil map) still lack the new agent role?"
  - "internal/shuttlecli/agent.go against open.go and the other session-starting paths: which preconditions and failure paths apply (pause refusal, run record and its cleanup, unawaited-session lifetime, a start that fails or outlasts the button's command), and does agent handle each?"
  - "placeClause and the reedGeometry inputs: for every worktree a task, beam or prime button can launch in, are NameSlug, NameShortname, HubPath and ParentName what the prompt claims? In particular, can a worktree with an empty slug be something other than the prime, or a slug-bearing one serve no board entry?"
  - "The name path from button to refusal: follow {answer} through reed's substitution, quoting and any check reed makes itself, into roleFromFlag. Do names with spaces, upper case, leading dashes or shell characters behave as the ask text says, and does what the client actually displays on a refusal show the rule? Does any other consumer of open --role depend on the value being passed through unchanged?"
  - "Test economy and coverage: which added behaviour in internal/shuttlecli (the agent verb, name normalization in agent and open, the refusal message) is held by a test, in this diff or an existing one?"
  - "Docs and change description: does the repository's constraints document name docs, help text or skill files that a new shuttle verb, stencil, config role or package must update in this diff? Check _lym/landing/summary.md claims, including the Merge danger door and blast radius, against what shuttle, agentconfig and the button templates can now break."
---

Round 1 has no earlier findings.
The entries follow the board entry's two expectations: a name rule the operator can see, and an agent that starts with its context, instructions, guards and skills.
