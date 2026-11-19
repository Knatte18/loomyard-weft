# Coverage map uses: reader, reach note, failure triage and lym gate mutate

`testmap-reach-triage-mutate` · Task · feature, testmap, gate, burler, darn, webster, loom, shed, stencil, pattern · running · Parent-Align

Decision record: [Sketch](decision-testmap-reach-triage-mutate.md)

Filed from the Sketch `testmap-uses-design`.

Takes: the per-test maps and FuncKey of internal/testcover, the merge base and base file contents read through internal/fabricengine as the redundancy step reads them, and the existing mutant overlay, binary builder and runner
Delivers: D1 to D9, D11 and D12 in the record's build order: the reader and the reach note first, then failure triage, then the changed-line planner and lym gate mutate; D10 stays designed only and is built in a later darn, as the D11 board note asks
Exports: the package internal/testreach (new), the reach report step and reach.json, the reach_report rubric marker, the reach turn of the loomshed verify gate and the webster batch gate, the verbs lym gate reach and lym gate mutate, the changed-line planner of internal/testcover, and the gate.yaml keys reach_budget_min, mutation_max_mutants and mutation_budget_min

The part changes these packages:

- internal/testreach (about 1100 changed lines)
- internal/gatecli (about 2000 changed lines)
- internal/gateslot (about 150 changed lines)
- internal/testcover (about 700 changed lines)
- internal/shedadapters (about 80 changed lines)
- internal/loomcli (about 120 changed lines)
- contracts/stencils/loom (about 20 changed lines)
- internal/loomshed (about 250 changed lines)
- internal/websterengine (about 300 changed lines)
- internal/weaverengine (about 80 changed lines)
- internal/runplan (about 60 changed lines)
- internal/gatefixer (about 150 changed lines)
- internal/shedrecipe (about 80 changed lines)
- pattern (about 30 changed lines)
