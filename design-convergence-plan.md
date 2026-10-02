Operator 2026-09-30: take a step back and plan for convergence, not keep accepting more bugs.
Observed today: nearly every hub intervention was in same-day new code (#319: approve nil dep, reject reading its file argument as run-id), and both passed tests built on fake deps.
Ingredients: (1) log every hub intervention and ly-drive repair per run with a class and cause, so interventions per run and new-vs-repeat failure classes are visible; (2) every fix at the source with a test that drives the real CLI wiring, not only fakes; (3) freeze periods with no new verbs or recipe rows, hardening only, until the intervention rate flattens.
Goal: the outside LLM steps in rarely, and for the task rather than for lyx.
## Model trial (from `model-spec-short-form`, operator 2026-10-01)

Try `fable[high]` for discussion and review, where design judgement matters (refusal-audit's holes came from opus Discussion and opus reviews; a Fable review found the design problem), and keep sonnet for implementation.
Change it only between runs, and measure review rounds, PR rejects and interventions before and after.