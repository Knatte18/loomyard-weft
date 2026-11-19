# Merge danger in descriptions, tautological-test review, lym glossary

Three changes to stencils, review rubrics and validation, bundled into one task.

## Merge danger in every change description

A change description now ends with a `## Merge danger` section:
**Door** is `one-way` when a revert of the landing commit does not undo the effect, and `two-way` otherwise;
**Blast radius** is one word naming what breaks if the change is wrong.
`contracts/specs/loom-final-summary-spec.md` defines the section, including what makes a door one-way:
a format another binary reads, a config reinterpretation `hubreconcile` applies, deleted data, anything pushed outward.

- `summaryparser.ValidateDescription` refuses a body without the section, a door other than the two words,
  and a blast radius that is not one word.
  `lym loom validate-description` and every gate that checks a description inherit this.
  `Publish` and `Finalize` do not check it.
- The darn write stencil and the landing Describe stencil ask for the section.
  The orch's beam role text asks for it too.
- `lym beam describe --draft` ends the draft with the section, its two values left as placeholders.
  The validator refuses both, so a draft recorded unedited is refused.
- The Code-Review rubric gains a line that checks the door against the diff:
  a `two-way` call is a finding when the diff changes a format another binary reads, changes what a config setting means,
  deletes data or pushes anything outward.
  A blast radius that names less than the diff can break is a finding too.

## Tautological tests in Code-Review

The rubric's Test economy list gains a bullet:
a test whose expected value is computed by calling the code under test, a helper it shares or the same formula
is a finding that names the test and the independent value it should assert.
A mutation survivor whose covering test recomputes its expected value is evidence for that finding.

The matching `scribe:testing` rule is not part of this change.
Scribe is its own repository, so that rule is filed on the scribe hub's board.

## A shared glossary

`internal/heddleengine/glossary.md` holds one line per lym term: its meaning and the words never to use for it
(an entry is never a "kort" or a card; a decision record is never a "logg"; a recipe is never a "profil").
It is loaded only where the drift happens:

- The orch gets it through a shipped heddle entry, `lym-glossary`, aimed at the `orch-prompt` target.
  `heddleengine.Glossary` returns the text, and the entry is added after every entry of `shipped.json`.
- Discussion gets it through a new `glossary` marker and an "lym's words" section in the Discussion stencil.
  Discussion calls out a term in the entry that conflicts with a line,
  and records a term it settles as a decision that gives the glossary line.
- It reaches no other agent, and nothing reaches it through `claude-prompt` or `agents-file`.

There is no lint: the drift was in chat with the operator, which a lint never reads,
and words such as "card" and "log" are right in other senses.
The `nicknames` entry stays for this hub's own nicknames.
`docs/reference/weaving-glossary.md` points to the glossary file.

## To check by hand

- `mergeDangerSection` in `internal/summaryparser/description.go` skips fenced code, so a fenced example is neither the section nor a heading that ends it.
  A review flagged that the scan ignored fences; the code now tracks the fence's marker and length.
  Try a body with a fenced `## Merge danger` example and a nested longer fence.
- Read the glossary lines in `internal/heddleengine/glossary.md` for accuracy; no test checks their meaning.

## Merge danger

**Door:** two-way
**Blast radius:** descriptions
