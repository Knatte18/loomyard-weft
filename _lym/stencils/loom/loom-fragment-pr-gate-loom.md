<!-- This is loom's pr-gate measure: what the alignment agent judges a loom run against at pr-gate, and what counts there as a scope deviation.
     The loom wiring of internal/alignment tells it, and it fills loom-template-alignment.md's pr_gate_measure marker.
     internal/stencil's StripLeadingComment removes this leading comment before the fill.
lyx-stencil: sha256=03bb47175d7668fee844949f768c015f64036e24bdc8b9383acf02e3ebe2fc86 build=0850f76e44d4328c27ab16d0bea13c08c7e0eb8e time=2026-10-14T16:52:39Z -->
Judge the run against its board entry's goal and the decisions recorded in the decision record.
A scope deviation is a change the goal does not call for, a goal point the change leaves out, or a recorded decision the change breaks.
