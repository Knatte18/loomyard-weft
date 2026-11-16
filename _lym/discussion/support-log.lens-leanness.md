```yaml
lens: leanness
outcome: reported
```

Task: review the decision record and support log for content to cut or move (leanness lens).

**MEDIUM — D2, the list of sweep hits.**
D2 names the stencils the sweep found ("the darn make-change fragments, the loom discussion, plan and parts stencils, raddle, twill…").
It is a snapshot from one subagent, it goes stale with any stencil change, and the plan derives it fresh.
The second sentence of D2 should be cut.
Keep the completeness that matters: the categories of terms the scan covers are already implied by D2's sentence translating each to Codex terms.
Once the list is gone, the first bullet of `## Notes for the plan writer` ("rerun the sweep… rather than trusting that list") has nothing to refer to, so cut it too.

**MEDIUM — `## Open risks`, first bullet (`developer_instructions` length).**
This is a design risk the writer could settle by reading how reed starts the Codex launch line: as an argv command, or as text typed into the pane.
Left as it is, it hands a fact the design rests on to downstream, which the rubric's "deferred research question" bullet flags.
Settle it and move the reasoning to the support log, or turn it into a bound in D5.
Open risks is meant for questions only the parent can settle.
The second bullet (plugin hooks reading Claude-only environment variables) already restates D8's bound, "left out with a warning". It should be cut, or folded into D8.

**LOW — `## Auto-mode assumptions` restates decisions.**
- "No exemption table" repeats D1.
- "session's model" and "default of 16" repeat D6.
- "orch out" repeats Scope.

Say each once: keep only the reasoning ("16 sits above the widest fan"), or point to the decision ids.

**LOW — D8 and D9, the bounds that check this machine.**
"Neither the operator's global file nor this repository holds one" and "none exists on this machine" pin facts about this particular machine.
They justify the bound but are not part of it.
Move them to the support log's ledger, Q5, and keep only the bound itself.

**LOW — D8 rationale, the last sentence (scribe hook example).**
It is an illustration, not a requirement.
It belongs in the support log's Interview turn 1.

**LOW — D2 rationale, "so the bench's darn writer read Agent-tool wording".**
This is history.
The fact that engine blocks exist but only assist seats use them is enough.

**LOW — D6, the `default_subagent_model` fact.**
No decision uses this key; D6 deliberately leaves the model unset.
Cut it from the facts.

**LOW — Scope "In" duplicates the list of decisions.**
Each bullet maps one-to-one onto D1–D11.
Scope could be one line pointing to the decisions plus the "Out" list, which carries the real content.

**LOW (not leanness) — D2's scan term list is undefined.**
The "closed list of Claude-only terms" is referenced but never given.
After cutting the hit list, D2 should name the term categories the scan holds: Agent tool, `subagent_type`, fork subagents, Skill/Read/Edit/Write tools, AskUserQuestion, "system prompt", Bash tool `timeout`.
That gives the plan writer a definite list.

No further tool calls made.
