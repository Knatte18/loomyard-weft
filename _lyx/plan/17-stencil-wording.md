# Card 17 — stencil-wording

**Prosa:**
- `contracts/stencils/edit/edit-directive.md`
- `contracts/stencils/loom/loom-rubric-webster-review.md`
- `contracts/stencils/pattern/pattern-directive-implementer.md`
- `contracts/stencils/pattern/pattern-directive-orchestrator.md`
- `contracts/stencils/pattern/pattern-directive-designer.md`
- `contracts/stencils/pattern/pattern-directive-review-fix.md`
- `contracts/stencils/pattern/pattern-directive-judge.md`
- `contracts/stencils/webster/webster-prefix-fork.md`
- `contracts/stencils/webster/webster-template-master.md`

**Intent:**
The remaining wording decisions, each pointing at the spec that owns a shape rather than restating it (`PATTERN-producer-pointer-rule`).

- `edit-directive.md` (D1.4) states the one exception, once: a Webster implementer working a `Rename` card's symbol pair runs the rename tool its prompt names; no other role and no other card runs a script or tool that rewrites file content beyond what the directive already allows.
  It states the bound in the directive itself: only commands the prompt's `rename_tools` list names, only on a `Rename` card's symbol pairs, only in a Webster fork or recovery strand, and the shell guard's rows still apply to the tool.
  `PATTERN-edit-directive` keeps this file the rule's only statement.
- `loom-rubric-webster-review.md`:
  - D1.9: the per-card mechanical check for a `Rename` group checks the outcome the diff shows, no reference to the old name left and the symbol's doc-comment opening updated, never that a tool ran.
  - D2.5: the review reads the commit messages of its range (`git log` over the same range) as well as the diff; a difference from the plan that a `Pattern-Departure:` trailer explains is judged on that reason, and holds when the named entry, read in its declared new text where the directive's declared changes list it, forbids the card's way; an unknown entry or one that does not forbid it is a finding.
    A trailer excuses only the difference it names.
  - D5.7: a new `Also flag` bullet checks that the branch's `PATTERN.md` and `pattern/` diff carries each entry the constraints section's declared changes list, its add, change or remove with the declared text, and raises a BLOCKING finding for each declared entry the diff does not land; it reads the declarations rendered into its directive, never the plan's section, so a bullet appended after the plan gate is checked too, and raises nothing when every declared entry lands.
- `pattern-directive-implementer.md` (D2.2): "say so in your report" becomes: state the conflict and the entry where your output records its reasons, a `Pattern-Departure:` trailer for a commit you make (see `loom-plan-spec.md`'s `## Pattern-Departure trailer`) and `## Shared Decisions` for a plan.
- `pattern-directive-orchestrator.md` (D2.2): its bullet on the forks it spawns says a fork that departs from its card because an entry forbids the card's way records it as a `Pattern-Departure:` trailer in its commit, as `loom-plan-spec.md` defines, and the digest's `departures` carries it.
- `pattern-directive-designer.md` (D5.1): the designer lists in the decision record's `## PATTERN changes` every entry its design adds, changes or removes, in the shape `loom-discussion-spec.md` defines, and makes no PATTERN edit itself, since the edit lands in a Webster card.
- `pattern-directive-review-fix.md` and `pattern-directive-judge.md` (D5.4): the BLOCKING rule and the never-downgrade rule each say an entry the declared changes list is judged by its declared text.
- `webster-prefix-fork.md` (D3.1): the `await-batch` takeover sentences go; once the report is written the fork's turn ends and the Master is woken by the fork's completion notification; the ban on running any `lyx webster` command stays; its banner states the spawn instruction the Master sends, the one `webster-template-master.md`'s loop step 2 quotes.
- `webster-template-master.md`:
  - D3.2: the self-identification rule keys on the opening both of the Master's spawn prompts share, "You are an implementer fork", in place of the "Read this file and follow it exactly" instruction the Master never sends, and its `await-batch` mention goes.
  - D3.3: the plan-drift paragraph says the run resumes once the plan is fixed: under loom the stuck run is routed to plan review, which edits the plan; standalone, the operator edits the plan and runs `rebaseline` or `restore-plan`.

All static prose; the existing marker and flag tests still read these files, and no test reads the wording.
