# Support log — motif-raddle-recipe

## Interview

The session ran autonomously, so each turn below is a self-made pick after reading the board and the code.

### Turn 1 — Board and exploration

Read the task, `motif-before-tapestry`, `darn-cost-measure`, `discussed-task-split` and `lym loom peers`; no `Decision for motif-raddle-recipe:` note exists.
Four explore subagents mapped the recipe registry and blocks, the raddle flow, the Burler ring and arbiter, and the board release and policy text.
Findings that shaped the design: the loom contract allows one record row; `finishesAsSketch` already keeps a recipe with a RaddleFile row from making a Sketch; `SplicePart` copies every Added after Discussion entry into every part; landing prunes done entries; dobby holds a released task until its dependencies are done; no project-wide release exists; the proposal checks refuse a filed slug equal to the Sketch's.

### Turn 2 — Recipe shape (D1)

Auto-pick: Motif block, then Raddle-Write, the split perch and Raddle-File, with no Record-Write row.

### Turn 3 — The perch (D2, D3, D4)

Auto-pick: a Bouncer ring with a cold fixer, a new raddle rubric, and a `perch` mode chosen by the recipe's rows.

### Turn 4 — Ends of the run (D5, D6, D10)

Auto-pick: one drafted task, or the arras entry turned into the project in the filing write; shape `beam` refused; dependents re-pointed at the filed work.

### Turn 5 — Release and record (D7, D8, D9)

Auto-pick: a project release releases its draft members; the project keeps the whole record; the shipped text names `arras`.

### Turn 6 — Lens review

Every lens returned a report.

- leanness 1 (D1 registration list): fixed, list dropped; `relevant.md` carries the files.
- leanness 2 (D2 rationale argues a rejected alternative): fixed.
- leanness 3, board-scope 3 (Scope says raddle's shape choice is unchanged): fixed, Scope names D5's limit; the shape Open risk dropped.
- leanness 4 (Auto-mode assumptions restate decisions): fixed, cut to the assumption and its id.
- leanness 5 (Sketch point twice): fixed, D1's sentence dropped, the Notes bullet kept.
- leanness 6 (`ClearMotifRun` twice): fixed, folded into D6.
- leanness 7 (Scope Out bullets): fixed, `cartoon` bullet cut, `default_recipe` kept with its reason.
- leanness 8 (`slack-v2` in D3 and D7): fixed, D3's rationale states the mechanism.
- board-scope 1 (decision class: "raddle fixes what it finds" reinterpreted): kept D2 as an auto-pick, grounded in Rejected alternatives.
- board-scope 2 (the project lists the order): fixed, D6 sets the project's `order`, with a criterion.
- interactions 1, existing-paths 5, bound-breaker 4, walkthrough 5 (perch re-entry after a Raddle-File return): fixed, D2 re-arms the perch with a fresh cycle and full budget, bounded by Raddle-File's halt.
- interactions 2 (two arras signals): fixed, D4 records a `perch` mode every check site reads.
- interactions 3, bound-breaker 2 (D8's run-lock claim false; beam and done projects): fixed, D8 refuses a project with a run status or `done`.
- interactions 4, bound-breaker 5 (status skip keyed on the body): fixed, D6 keys the skip on a filing mark among the run's records, cleared with them.
- interactions 5, walkthrough 2, existing-paths 4 (decision add at or after Raddle-File): fixed, D7 refuses from Raddle-File on, by rows, with ways forward.
- interactions 6, bound-breaker 3, walkthrough 1 (dependents of the arras entry): fixed, new D10.
- interactions 7 (D3 re-derives D5's check): fixed, D3 judges only whether one task or a family fits.
- bound-breaker 1 (failed filing removes the entry's record): fixed, D6 restores the record, with a criterion.
- bound-breaker 6 (admission by caller): fixed, D6 admits by shape while the entry's run status names `Raddle-File`.
- existing-paths 1 (copied `carry_over` and `approve_seam`): fixed, D2 sets neither.
- existing-paths 2, walkthrough 4 (new commit seam): fixed, D2 sets no `commit_seam`; Raddle-File's commit covers the proposal.
- existing-paths 3, walkthrough 3 (other release paths; filing write releasing): fixed, D8 names the verb as the only trigger, with a criterion.

## Rejected alternatives

- Composing arras as the Motif block plus the whole raddle recipe: two record rows break the loom contract.
- Re-entering Raddle-Write as the perch's fixer: a fresh Opus session rereads the board and the code each round, and it bypasses the Bouncer escalations the arbiter serves.
- An approve seam recording a hash of the proposal and the record for Raddle-File to match: a new Env seam for a narrow window, which every part's record already covers by carrying the post-Discussion entries.
- A raddle `commit_seam` on the Bouncer: a new Env closure, where Raddle-File already commits the raddle directory.
- Filing the family's project under a new slug and ending the arras entry done: the operator wants the project named after the arras entry's slug.
- Removing the arras entry and adding a project with the same slug: the same end state as the in-place change, with a moment where the run's slug has no entry.
- Keying the post-filing status skip on the project body's prefix: an operator body edit would drop it, and a later beam run under the slug would lose its status writes.
- Keying the post-filing status skip on the board's raddle store under `raddle/<slug>/`: no run clearing removes it, so a later beam run under the slug would lose its status writes.
- Refusing an arras start for an entry other entries depend on: re-pointing the dependents keeps the entry usable and the dependency true.
- Allowing shape `beam` in arras: creating the beam starts the family's runs, which takes the release from the operator.
- A dedicated `lym board release-project` verb: a null status on a project is the release the operator already writes for a task.

## Review rounds

_No rounds yet._

## Question ledger

- Resolved (auto-pick): the fixer is a cold Burler fixer whose model defaults to the raddle model, not Raddle-Write re-entered. Reason: arbiter integration and cost.
- Resolved (auto-pick): releasing a project releases every draft member at once; dobby orders them by `depends_on`. Reason: the orch proposed it and dobby already gates on dependencies.
- Resolved (auto-pick): the project keeps the whole decision record. Reason: raddle already stores it on a filed project.
- Resolved (auto-pick): shape `beam` and multi-part `tasks` are refused in arras. Reason: the operator named two ends, one task or a project.
- Resolved (auto-pick): the project takes the arras entry's slug by turning the entry into the project. Reason: a slug holds one entry.
- Resolved (auto-pick): the arras signal is a `perch` mode recorded from the recipe's rows. Reason: `PATTERN-shed-recipe-registry` forbids deciding by recipe name.
- Resolved (auto-pick): a goto into Raddle-Write from a perch that has not settled resumes the perch's recorded round instead of opening a fresh cycle. Reason: a Subject seam over the proposal is new machinery for a recovery path, and Raddle-File's checks and the draft status still hold.
- Resolved (auto-pick): Raddle-File halts for the parent when the run's entry is already a project not filed from its current proposal. Reason: a send-back would re-split onto a slug a filing already holds.
- Resolved (auto-pick): dependents of the arras entry move onto the filed work. Reason: a dependent left on a done entry starts before the work lands, and one left on the converted project never starts once its landed members are pruned.
