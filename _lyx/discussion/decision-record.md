# Plan gaps: inserting a card, unreachable prescribed refusals

## Goal

Close two gaps in how plans are written and reviewed (board task `plan-gaps`, issues #508 and #516):

1. An agent editing a plan has no sanctioned way to insert or remove a card, so one inserted card forces a hand rewrite of every later card file and an unsanctioned `rm` of the old names.
2. Plan review never checks that a refusal a card prescribes is reachable, so an unreachable refusal passes Plan-Review and surfaces only after its code is written.

## Scope

In:

- Two new `lyx loom` verbs, `insert-card` and `remove-card`, that insert or remove one card in the current plan and renumber the cards after it.
- Their refusals, as rows in `contracts/specs/refusal-spec.md`'s loom section.
- One sentence each in the Plan-Write stencil (`loom-template-plan.md`), the PR-Rework stencil (`loom-template-rework.md`) and the Plan-Review rubric's `## Write surface` section, naming the two verbs as the only way to insert or remove a card file.
- A reachability rule for prescribed refusals: a writer rule in `loom-template-plan.md` and the matching `## Also flag` bullet in `loom-rubric-plan-review.md`.
- Docs in the same commit: `internal/loomcli`'s `doc.go`, the lyx command index the `ly:cli` skill loads, and the numbering section of `contracts/specs/loom-plan-spec.md` where it says how numbering changes.

Out:

- Renaming a card's slug, moving a card to another position, or bulk renumbering.
- Any change to the burler stencils (`burler-step-3-fix.md`, the fix orchestrator, the fix-scope rules in `internal/burlerengine`); the tasks `fixer-critical-of-findings` and `burler-multi-llm` own those.
- A mechanical (Go) reachability check; reachability is judged by the plan writer and Plan-Review.
- Any change to the Webster-Review rubric.
- Any change to the edit directive or the agents' Bash deny list.

## Decisions

### D1 — Two loom verbs carry card insertion and removal

**Decision:**
`lyx loom insert-card <N> <slug> --intent "<one-line intent>"` makes card `N` a new card: every card numbered `N` or higher moves up by one, then the verb writes `NN-<slug>.md` holding only its title heading (`# Card N — <slug>`) and adds the Card Index line `N — <slug> — <intent>`.
`N` may be one past the last card, which appends with no shift.
`lyx loom remove-card <N>` deletes card `N`'s file and Card Index line, and every later card moves down by one.
Moving a card from number `M` to `M±1` rewrites exactly four things: its file name prefix, its `# Card M` heading number, the `M: ` prefix of an explicit `**Commit:**` line, and its number on its Card Index line.
The new card's body is written by the agent afterwards with Edit or Write; until it is, the plan gate reports the stub's missing fields, so a forgotten stub cannot pass.
Both verbs print a JSON envelope listing each moved card (old and new number and file), the created or removed file, and every line in the current plan's files that names a moved card's old number in prose (`card 8`, `Card 8`), as file and line, for the agent to check by hand.

**Rationale:**
A verb turns a twelve-file hand rewrite into one call, keeps the edit directive's "Edit or Write only" rule intact for content, and gives file removal one owner instead of a stated `rm` exception.
Prose cross-references cannot be rewritten safely (a mention of card 8 may mean the old or the new card 8), so the verb reports them and leaves them alone.

### D2 — The verbs' bounds

**Decision:**
The verbs are new edges that rename, rewrite and delete files, bounded as follows:

- They touch only the current generation's card files and the Card Index of `00-overview.md` under `planparser.PlanDir`; never `archive-*/`, `round-<N>/prior-generation/`, the overview's frontmatter, or any other overview section.
- They refuse a plan that does not parse, or whose Card Index and card files disagree (the `index-file-mismatch` or `card-numbering` checks), before changing anything, since renumbering from an inconsistent plan compounds the inconsistency.
- They refuse a position outside `first_card` to last card (remove) or `first_card` to last card plus one (insert).
- They refuse a position at or before the last card of the last batch webster's run record holds as begun, read the way `loomshed.ValidatePlan` reads it; with no run record, every position is open.
  This matches the frozen-card rule the Plan-Review rubric and `lyx webster rebaseline` already enforce.
- `remove-card` refuses to remove the plan's only card, since a cardless plan does not parse.
- `insert-card` refuses a slug the plan format's slug rule rejects.
- They never commit, never approve and never set `approved:`; the row's commit seam commits the plan directory as today, deletions included.

Each refusal names its way forward, has its row in `refusal-spec.md`'s loom section and is reached by a test (`PATTERN-refusal-way-forward`).
The verbs compute every new file name and content in memory first, then apply: moves in descending order for insert and ascending order for remove, so no two files ever claim one number, and the Card Index last.
A raw I/O failure mid-apply joins the loom section's grouped transient I/O row; its envelope lists the steps already applied, and the plan gate reports the partial state.

**Rationale:**
Frozen begun cards and the parseable-plan precondition are the two ways a renumber could corrupt run state; refusing both keeps the verbs as safe as a hand edit the gate would catch, and no less.

### D3 — Where the verbs live

**Decision:**
The plan-format rewrite (renumbering file names, headings, Card Index lines and commit prefixes) lives in `internal/planparser`, beside its existing writer `SetApproved`, since `planparser` is the plan's sole parser (`PATTERN-sole-parsers`) and owns the plan directory's paths (`PATTERN-lyxdirs-single-declarer`).
The two cobra verbs live in `internal/loomcli`, beside `validate-plan`, wired through `internal/cliwire` (`PATTERN-cli-cobra`, `PATTERN-cliwire-sole-wiring`).
The begun-card bound reads webster's run record through `websterengine`, as `loomshed.ValidatePlan` already does.

**Rationale:**
Loom owns the plan artifact's CLI surface (`validate-plan`); `planparser` owns its format.

### D4 — Agents are told through their own stencils

**Decision:**
`loom-template-plan.md`, `loom-template-rework.md` and the `## Write surface` section of `loom-rubric-plan-review.md` each gain one sentence: a card is inserted or removed only with `lyx loom insert-card` or `lyx loom remove-card`, never by renaming, rewriting under a new name or deleting a card file by hand.
The rubric is the channel for the Plan-Burler fixer, which receives the rubric in its round prompt; the burler stencils stay unchanged.
The sentences name the verbs and point at their help; they restate neither the bounds nor the envelope (`PATTERN-producer-pointer-rule`).

**Rationale:**
The rubric reaches both halves of the Plan-Review round without touching the burler stencils two other running tasks are editing.

### D5 — Prescribed refusals must be reachable

**Decision:**
`loom-template-plan.md` gains a writer rule: a card that prescribes a refusal, error return or guard branch names in its `**Intent:**` the input that reaches it after every upstream check passes, and the test that reaches it.
A branch that upstream validation already rules out is not prescribed: it is dropped, or, when it guards an invariant, it is a fail-loud wiring guard in `refusal-spec.md`'s sense, which takes no per-row test.
`loom-rubric-plan-review.md` gains the matching `## Also flag` bullet: a card prescribing a refusal with no named reaching input or test is a finding, and so is one whose named input an upstream check rejects; the finding names that upstream check.
A test that calls only the refusal's error constructor does not count as reaching it.

**Rationale:**
`PATTERN-refusal-way-forward` already requires a test that reaches each new refusal, and the code-quality rule forbids a code path the code's own guarantees prevent; a card that cannot name its reaching input violates both before any code exists.
Reachability depends on what upstream checks reject, which only reading the code answers, so it is a review judgment, not a mechanical check.
The writer rule and the rubric bullet are paired so the writer/reviewer symmetry note holds.

## Constraints

- `PATTERN-sole-parsers`, `PATTERN-lyxdirs-single-declarer`: the plan rewrite goes through `planparser`, and no new file spells `_lyx`.
- `PATTERN-cli-cobra`, `PATTERN-cliwire-sole-wiring`: the verbs are cobra subcommands of `lyx loom` with `audience`, `Short` and JSON errors, resolved through `cliwire`.
- `PATTERN-refusal-way-forward`, `PATTERN-no-denied-recovery`: every refusal names a way forward that is not a denied command, lands its row in the same commit, and is reached by a test.
- `PATTERN-producer-pointer-rule`: the stencil sentences point at the verbs and never paraphrase the plan format.
- `PATTERN-plan-generation`: the verbs act on the top-level generation only and never touch an archived one.
- `PATTERN-test-economy`, `PATTERN-test-speed`: the verbs' behavior is pinned at their public surface with table-driven tests at the lowest fitting tier; the test-budget file is listed under `Edit:` if a package's top-level test count passes its row.
- `PATTERN-comment-line-breaks` and the semantic-line-break rule for every `.md` edit.
- The edit directive is unchanged: content edits stay Edit or Write; the verbs are the only sanctioned file rename and removal in the plan directory.

## Auto-mode assumptions

- The verbs are flat `lyx loom` subcommands named `insert-card` and `remove-card`, matching the flat `validate-plan` sibling, rather than a nested `lyx loom plan` group.
- `insert-card` takes the Card Index intent through a required `--intent` flag, so the overview is complete after the call and only the card body is left to write.
- The inserted card file holds only its title heading; no field skeleton is written.
- Prose mentions of moved card numbers are reported, never rewritten.
- The Webster-Review rubric is left unchanged; plan-time reachability is the fix the board entry asks for.
- Reachability stays a review judgment; no Go check is added.

## Open risks

- `lyx webster rebaseline` may need every moved unbegun card named with `--card NN` after an insert during Webster; the envelope's moved-card list gives the numbers, but the plan should confirm how rebaseline treats a renumbered unbegun card.
- A plan-review reviewer may still miss an unreachable refusal whose upstream rejection lies several packages away; the rule makes the card state its reaching input, which narrows but does not close that gap.

## Acceptance criteria

- `insert-card` at a middle position moves every later card up by one (file name, heading number, explicit `Commit:` prefix, Card Index number), writes the new card's heading-only file and Card Index line, and the result passes the `index-file-mismatch` and `card-numbering` checks.
- `insert-card` at one past the last card appends with no moves.
- `remove-card` at a middle position deletes the card's file and index line and moves every later card down by one, with the same consistency result.
- A plan whose `first_card` is above 1 is renumbered within its own range.
- The envelope lists moved cards, the created or removed file, and prose lines naming a moved card's old number; those prose lines are left unchanged.
- Each refusal is reached and leaves the plan directory byte-identical: unparseable plan, index/file mismatch, position out of range, position at or before the last begun card per webster's run record, removing the only card, invalid slug.
- `archive-*/` and `round-<N>/prior-generation/` contents are untouched by both verbs.
- Each new refusal has its row in `refusal-spec.md`'s loom section with its way forward.
- The Plan-Write and PR-Rework stencils and the Plan-Review rubric's write surface name the two verbs as the only way to insert or remove a card.
- The Plan-Write stencil carries the reachability rule and the Plan-Review rubric carries the matching `## Also flag` bullet, including that a constructor-only test does not reach a refusal.
- `internal/loomcli`'s `doc.go` and the lyx command index describe both verbs.

## Notes for the plan writer

- `loomshed.ValidatePlan` in `internal/loomshed/gates.go` shows how to read webster's run record (`websterengine.LoadState`, `websterengine.DoneCards`); the begun-card bound wants the begun rather than the done set, matching `lyx webster rebaseline`'s "at or before the last begun batch's last card" rule in `internal/websterengine/rebaseline.go`.
- `planparser.SetApproved` is the existing plan writer to sit beside.
- The `index-file-mismatch` and `card-numbering` checks, and the `first_card` rule, are in `contracts/specs/loom-plan-spec.md`'s validation-checks and numbering sections.
- The Plan-Burler row's prose and target paths are in `contracts/recipes/loom-recipe.yaml`; the row's target is the whole `_lyx/plan` directory, and `CommitPlan` in `internal/loomcli/wiring.go` commits the whole directory, deletions included.
