---
round: 1
focus:
  - "internal/websterengine/planfixer.go and internal/webstercli/fixplan.go: does each D6 refusal (no record, equal findings to an earlier plan fixer, two fixers for the batch) fire, name a way forward, and get reached by a test? Is the findings comparison correct against the record format?"
  - "planfixer.go attach and await path: does a re-entrant call attach to a non-terminal record instead of spawning, and does a dead or timed-out fixer yield verdict plan_writer rather than a hang or error?"
  - "D7 HEAD and clean-tree check: is the HEAD recorded at spawn and compared after the fixer ends, and does a moved HEAD or dirty tree turn only a fixed verdict into plan_writer with a warning?"
  - "state.go, runlevel.go, stopper_test.go: is the plan-fixer record loaded absent as nil, stopped and marked dead on run respawn, stopped on reset, and kept out of recordedSessions?"
  - "verifyfixer.go and coldfixer.go refactor: does extracting shared cold-fixer code leave fix-verify behavior unchanged?"
  - "contracts/stencils/webster/webster-template-plan-fix.md and its render in render.go: does the prompt carry the conflict record path, plan directory, reply path, parent and edit directives, and the rebaseline and restore-plan instructions, point at the plan spec rather than restate it, and forbid git and other webster verbs? Is it read through stencilstore at call time?"
  - "webster-template-master.md: does the plan-conflict rung run fix-plan backgrounded, re-begin on fixed, end stuck on plan_writer, dead fixer or refusal, and keep plan_drifted stuck, without dropping other rules?"
  - "Scope against the board entry and record: are docs updated (websterengine and webstercli doc.go, webster-spec, llm-model-spec, overview) and no change made to the loom recipe, shed adapter or plan-review instructions?"
  - "Test economy: do fixplan_test.go and planfixer_test.go duplicate each other's coverage, and is any added behavior uncovered?"
---

Seed focus drawn from the decision record's decisions D1-D7 and acceptance criteria, mapped to the diff's files.
