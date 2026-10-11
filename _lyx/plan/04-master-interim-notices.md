# Card 4 — Master treats an interim fork notice as no completion

**Edit:**
- `contracts/stencils/webster/webster-template-master.md`
- `contracts/stencils/claims_test.go`

**Intent:**
Decision record D5: with gates in the foreground an interim fork notice arises only from a Bash call the tool backgrounds at the one-hour ceiling, and Merriam needs a safe reading of one.
Shuttle no longer lists a fork that sent an interim notice as outstanding, so step 4 as written would send that notice to `record-batch`, which classifies the missing report as a failed batch while the fork still works.

`webster-template-master.md`:
- The loop's step 4 gains, after its first sentence: "A fork notice whose message says the fork is still waiting on a command is not its completion: end your turn with no tool call, and its completion notification starts a later turn."
  Every other fork notice still goes to `record-batch`.
- Under `## A gate failure: spawn one fixer fork`, step 4 gains: "A notice from the fixer whose message says it is still waiting on a command is not its completion: end your turn with no tool call, and leave `{{.outcome_path}}` and `{{.summary_path}}` as they are."
- After "Spawn at most one fixer fork per gate message;", the section adds: "A `Gate findings recorded at …` message that arrives while an earlier fixer has sent no final reply spawns no second fixer: end your turn with no tool call."
- The FIRST paragraph's "ending a turn with no running background work and no outcome file ends the run with nothing done" is corrected to shuttle's behaviour: "a turn end with no running background work and no outcome file is held, with a hold notice to your parent, until a later message starts your next turn or the run's timeout, liveness check or idle kill ends it".
  The parenthetical naming the safe turn ends, the backgrounded fork and backgrounded `recover-batch`, stays.

Bound (D5): this edge lets a notice skip `record-batch`, or the fixer's rewrite, on Merriam's reading of model-written text.
A batch-loop notice misread as interim leaves the turn end held, with the parent's hold notice showing the stall, until `master_timeout_min` or loom's step idle kill.
A fixer's interim notice makes shuttle read Merriam's turn end as a gated arrival, and the dirty-tree gate failures spend `verify_gate_attempts`, which bounds it;
the no-second-fixer line keeps two fixers off the tree at once.
The template names no config key for these bounds.

`claims_test.go`, in the `webster-template-master.md` entry: a `wantAll` pins the step-4 interim sentence;
an `inSection(websterGateSection, …)` `wantAll` pins the fixer interim sentence and the no-second-fixer sentence;
a `wantAll` pins the corrected held-turn-end clause, and a `wantNone` pins that "ends the run with nothing done" is gone.
Every existing row keeps passing: "No operator answers a question here" and the backgrounded fork and `recover-batch` wording are untouched.

**ImpactSummary:** Merriam's prompt gains three reading rules for interim notices and a corrected statement of an idle turn end; no Go code changes.

**Commit:** 4: Master reads an interim fork notice as no completion
