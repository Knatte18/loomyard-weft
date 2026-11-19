# Merge danger in descriptions, tautological tests, and lym's glossary

Three small changes from the survey of Matt Pocock's skills repository (`docs/research/survey/pocock-skills-2026-11-19/report.md`).

## Merge danger in every change description

A change description now ends with a section that says what a mistake would cost:

```markdown
## Merge danger

**Door:** two-way
**Blast radius:** stencils
```

The door is `one-way` when reverting the landing commit does not undo the change's effect.
Examples are a format another binary reads, a config reinterpretation, deleted data, or anything pushed outward.
The blast radius is one word naming what breaks if the change is wrong.

- `summaryparser.ValidateDescription` refuses a non-empty body that lacks the section, has a door other than `one-way` or `two-way`, or has a blast radius that is not one word.
  Each case has its own finding (`no-merge-danger`, `merge-danger-door`, `merge-danger-blast-radius`).
  The section runs to the next heading of level one or two.
  Each labelled line may be a list item, and its value may be in backticks.
- `ValidateDescription` backs `lym loom validate-description`, the Describe row's gate, the darn writer's description gate, `lym beam describe` and the pawl ready check.
  All of them refuse such a description.
  `Publish` and `Finalize` still only parse the description.
- The final-summary spec defines the section.
  The darn writer and Describe stencils ask for it, and so do the orch's beam rules.
- `lym beam describe --draft` ends the draft with the section and placeholder values.
  The check refuses those placeholders, so the orch has to fill in both values before the description is recorded.
- The Code-Review rubric gains a **Merge danger** bullet.
  It checks the door against the diff and treats a `two-way` call with the most suspicion.

## Tautological tests in Code-Review

The rubric's Test economy list gains a bullet.
A test whose expected value is computed the way the code computes it is a finding: it calls the code under test, a helper the code shares, or the same formula.
The finding names the test and the independent value it should assert instead.
The mutation report section adds that a survivor is evidence for this finding when its covering test recomputes the value.

## lym's glossary

`internal/heddleengine/glossary.md` holds one line per lym term: what it means, and the words never to use for it.
It covers entry, decision record, recipe, Sketch, run, card and other terms.

- heddle ships the glossary as the `lym-glossary` entry, aimed at the `orch-prompt` target, so it reaches orch sessions only.
- The Discussion stencil gets the same text through a `glossary` marker that `DiscussionTable` fills from `heddleengine.Glossary()`.
  The stencil tells Discussion to raise any term the entry uses against the glossary.
  When the design settles a term, Discussion records its glossary line as a decision.
- Writers, fixers and reviewers do not get the glossary.

## Design decisions

- **The glossary ships in the binary**, so every hub's orch and Discussion get it.
  This was the open question the entry left for Discussion.
  The hub's `nicknames` heddle entry stays as it is.
- **The glossary is kept apart from `shipped.json`.**
  It is a markdown file embedded beside `shipped.json` and added to the shipped layer at package load.
  That way a term is edited as markdown rather than inside a JSON string.
  `PATTERN-heddle-instructions`'s background now names it.
- **Discussion cannot write the glossary.**
  It may write only its three files, so it records a settled term as a decision, and the run's change adds the line.
- **Discussion reads the shipped glossary text.**
  A hub's `lym_override` of `lym-glossary` changes what the orch sees, but Discussion still gets the shipped text.
- **No `ly:vocabulary` skill.**
  The entry made it optional, and the glossary already reaches the two roles that need it.
- **The `scribe:testing` rule is left out**, as the record's parent entry of 2026-11-19 decides.
  scribe is its own repository and hub, so the parent files that rule on the scribe hub's board.

## To check by hand

- Beam descriptions recorded before this change have no Merge danger section.
  The pawl ready check and `lym beam describe` now refuse them until the orch adds one.
- Read the glossary's lines and correct any meaning or banned word the operator would put differently.

## Merge danger

**Door:** two-way
**Blast radius:** descriptions
