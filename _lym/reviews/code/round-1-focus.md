---
round: 1
focus:
  - "internal/beamcli/dobbydeps.go, runAutonomyStamps, against D1's 'the dobby posts the refusal once to its orch per episode': when is a refused value noticed, and what opens and ends an episode? Trace a run that builds no stamped mail, a dobby restart, several runs or concurrent posters, a failed post, a read that fails for a non-refusal reason between two refused reads, and holding the mutex across the post. Check which of these the tests pin."
  - "Every path that checks or reads beam.autonomy (the lookup, LoadConfig, lym config beam --set, CheckLocalValues for run_settings, LookupWithSource and its consumers, env-expanded values, the board mode map, reinterpretations): do all take the three levels alike, refuse other values, and read true and unattended as on wherever only on/off is asked? Search outside the diff for readers that still compare to a boolean or the text true, and for help text or comments that still describe two values."
  - "The unattended level as an orch meets it: the prompt line in internal/orchengine/prompt.go, the halt stamp, the template comment, and contracts/orchrole/ (role-autonomy.md, orch-role.md), which key on `autonomy: true`. Settle whether what the diff says and points at is consistent with D1 and with the amended board entry's Delivers and Scope, and whether the change description's account of what is not yet covered is accurate."
  - "New or changed messages and comments against the constraints: does the refusal post line in dobbydeps.go and the refusal in checkTaskSetting name the key as PATTERN-config-key-names requires, and does a test reach the way forward the refusal names (PATTERN-refusal-way-forward), including setting the value unattended through that command?"
  - "_lym/landing/summary.md: check the Merge danger door and blast radius against what the diff can break (a new config value an older binary refuses, the halt stamp and board mode texts other readers consume), and check each claim under What changes and Tests against the diff."
  - "Tests changed in the range: is any existing assertion widened or removed without a Loosened-Check trailer, and is any behaviour the diff adds (the Slack hook, loom drift and pawl seams reading unattended as on, the orch level mapping) left uncovered?"
---

Seed focus for round 1. Each entry is a question to settle, not a verdict.
