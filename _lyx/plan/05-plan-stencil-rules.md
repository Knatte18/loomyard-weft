# Card 5 — plan-stencil-rules

**Prosa:**
- `contracts/stencils/loom/loom-template-plan.md`
- `contracts/stencils/loom/loom-template-rework.md`
- `contracts/stencils/loom/loom-rubric-plan-review.md`

**Intent:** Tell the agents about the verbs card 4 added, and pair the reachability rule between the plan writer and its reviewer, each addition one or two sentences that point at the verbs and restate neither their bounds nor their envelope.

`loom-template-plan.md`, "On-disk layout" subsection, after the no-gaps sentence: a card is inserted or removed only with `lyx loom insert-card` or `lyx loom remove-card`, which renumber the later cards and whose `--help` says what they do, never by renaming, rewriting under a new name or deleting a card file by hand.
`loom-template-rework.md`, Step 3's bullet list, the same sentence for a card of the new generation.
`loom-rubric-plan-review.md`, `## Write surface`, the same sentence addressed to the fixer, so the Plan-Burler row reads it in its round prompt and the burler stencils stay unchanged.

The reachability rule, in `loom-template-plan.md` directly after the attack-surface paragraph: a card that prescribes a refusal or guard branch names in its `**Intent:**` the input that reaches it after every upstream check passes, and the test that reaches it; a propagated raw I/O error that joins its section's grouped transient row in `refusal-spec.md` is exempt; a branch that upstream validation already rules out is not prescribed, so it is dropped, or, when it guards an invariant, written as a fail-loud wiring guard in `refusal-spec.md`'s sense, which takes no per-row test; a test that calls only the refusal's error constructor does not reach it.
The matching `## Also flag` bullet in `loom-rubric-plan-review.md`, bold-led like its neighbours: a card prescribing a refusal or guard branch with no named reaching input or test is a finding, and so is one whose named input an upstream check rejects, the finding naming that upstream check; the same scope and the same raw I/O exemption, including that a constructor-only test does not reach a refusal.
The writer/reviewer symmetry note already in the rubric binds the pair, so neither side says what the other does not.

Every sentence is one per line, and no sentence names a command the agents' settings deny.
