# Card 28 — refusal-spec

**Create:**
- `contracts/specs/refusal-spec.md`

**Prosa:**
- `contracts/specs/webster-spec.md`

**Intent:**
Write the durable refusal table D3 specifies, the lookup ly-drive and the operator use for an escalation's way forward.

`refusal-spec.md` opens with a status line (Contract, kept on landing), then states D1's definition of a way forward and D2's four classes with their dispositions, once, as the file's own reference.
One `##` section per bound module: webster, shed, loom.
Each section is one table with the columns Refusal, Trigger, Class, Way forward.

- One row per refusal reachable through the module's verbs, with the class D2 assigns: every row cards 5–27 named, each audit class with its D4 severity and its disposition (warning, failed batch, or run-exit demotion), and the new `rebaseline` and `goto` refusals.
- The grouped rows each sweep card listed become one wiring-guard row and one transient raw-I/O row per section, naming what they cover.
- The Way forward cell quotes the verb, git command or transition the message names, so the cell and the message cannot disagree.
- A validation verb's findings envelope (`lyx webster validate`, loom's `validate-*` verbs) is that verb's verdict on its artifact, not a refusal; the loom and webster sections say so in one sentence rather than adding rows.

Out of scope, stated in one closing paragraph: landing, batten, orch, fabric, burler and board refusals, which a later audit adds as its own section, and the fast-forward merge-in limit, whose message already names a git recourse.

The file is plain markdown, not registered in `contracts/specs/specs.go`.
It is written with semantic line breaks; table cells stay on one line.

`webster-spec.md` gains a See-also entry linking `refusal-spec.md` as the table of webster's refusals and their ways forward, restating no row.
