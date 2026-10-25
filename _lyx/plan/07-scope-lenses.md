# Card 7 — scope-lenses

**Edit:**
- `internal/lenses/template.yaml`
- `internal/lenses#TestConfigTemplate_DecodesAndIsSelfConsistent`
- `internal/lenses#TestLoad`

**Intent:** D5 and D6: the Discussion-Review perch's round-1 fan carries the scope check the alignment agent made.

`internal/lenses/template.yaml`:

- The `review-discussion` fan becomes `interactions`, `existing-paths`, `bound-breaker`, `walkthrough`, `goal-scope`, `board-fit`; `twill-discussion` and `discussion` keep their members and resolve the rewritten texts.
- `goal-scope` and `board-fit` are rewritten from emphasis lenses into procedures that end in their own list, in the voice of the other procedure lenses, without the emphasis line, and naming no output format of `lyx loom peers` (`PATTERN-stencil-ownership`).
  Each read set is at most six tool calls, its decision-note reads included, within the six calls `burler-cluster-fan.md` gives a fork past its manifest.
- `goal-scope`: own the record's fit with its board entry.
  Read `lyx board list` once and `lyx board get` for this task, then up to four board notes titled `Decision for <slug>:` for this task, newest first, by `lyx board get`.
  Check the record against the entry: a goal point left out, work that belongs to another entry, a direction the entry rules out, unlicensed work, or a deferral; each is an ordinary finding for the fixer when the writer could settle it.
  Report doubt about what the task is for as a finding of class `decision`, phrased as a question for the run's parent.
  Its list: each goal point of the entry and whether the record meets it, and each decision note read and any the list names that it could not reach.
- `board-fit`: own the record's fit with the rest of the hub.
  Read `lyx board list` and `lyx loom peers`, then up to four, nearest first, of: `lyx board get` for a touching entry or for a `Decision for <slug>:` note for this task, `lyx loom peers show <slug> --record decision` or `--record plan` for a close peer, and read-only `git show` of a parent commit `lyx loom peers` lists.
  It names this task's decision notes from the `lyx board list` titles it already holds, and spends one of its four reads on a note before raising a parent question that note's title touches.
  Report as a `decision` finding, phrased as a question for the run's parent: the entry plainly outdated, because landed work overtook it or another board entry or run contradicts it; a design conflict with another run, where both change the same behaviour or the same PATTERN entry in different ways, or a decision in one contradicts or duplicates one in the other; or a landed parent commit that contradicts one of the record's decisions, or already makes one in a way the writer cannot settle by citing or dropping the decision.
  A landed commit that already makes a decision the writer can cite or drop is an ordinary finding.
  Its list: each entry, peer and parent commit checked, with overlap, conflict or none, and each decision note read and any it could not reach.
- Both texts also state: a board note titled `Decision for <slug>:` for this task, and the record's `Added after Discussion` entries, are parent decisions and win over the entry body; a text-only overlap (the same file or package at different places) and a shared bookkeeping line (a module-table line, a PATTERN index line) are no finding; a question such an answer already settles is no finding; an ordinary misalignment the writer could settle is an ordinary finding.

Tests:

- `TestConfigTemplate_DecodesAndIsSelfConsistent` wants `review-discussion` at six entries in the order above; its sentence check gains, for `goal-scope`, `Decision for <slug>:` and `class \`decision\``, and for `board-fit`, `lyx loom peers` and `spends one of its four reads on a note`, so a rewrite that drops the parent route or the read bound fails.
  The emphasis-line exemption already follows from the lenses joining a `review-` fan.
- `TestLoad` gains a case whose hub `lenses.yaml` defines `goal-scope` with its own text and asserts `ResolveFan` over `review-discussion` returns that text for `goal-scope` and the template's for `board-fit`, the criterion that a tuned lens keeps its own text.
  Reconcile reports nothing for a name the template holds, so such a hub runs its old text unreported until it drops the entry, as D5 records.

What it lets through: nothing new; the lenses add findings, and a `decision` finding stops the run only once card 12 routes it.
The bound is the read set: a fork can miss an entry, peer, commit or note past its reads, and its list names what it read and could not reach.

**ImpactSummary:** One fan gains two lenses and two lens texts change; the readers are the perch's round-1 fan, `twill-discussion` and the dormant `discussion` fan.
