# Card 1 — Plan cards name test members as glyphs

**Edit:**
- `contracts/stencils/loom/loom-template-plan.md`
- `contracts/stencils/loom/loom-rubric-plan-review.md`
- `contracts/stencils#wordingClaims`

**Intent:**
Decision D1: a plan card names an existing test function, fake or helper it changes as a member glyph, and the review rubric checks it.

In `loom-template-plan.md`, extend the paragraphs under "Each `NN-<card-slug>.md`" that start "An implementation card lists the implementation's member glyphs under `**Edit:**`" and "`**Edit:**` lists the member glyphs the card changes".
Add these rules:
- A card names an existing test function, fake or helper it changes as a member glyph, for example `internal/x#TestFoo` or `internal/x#fakeClock`.
- A plain test-file path stays only for a test file the card adds tests to without changing an existing member.
- A new test function in an existing file is a `**Create:**` declaration with a `plan:` handle.
- A new test file is a `**Create:**` path.

Keep the sentences the existing `wordingClaims` rows pin: "`**Edit:**` lists the member glyphs the card changes", "a file path only for a file with no symbol to name" and the skeleton's "- `path/to/list_test.go`" bullet.

In `loom-rubric-plan-review.md`, extend the "**Member glyphs on an `Edit` card.**" bullet to test files.
An `Edit` card that lists only a test-file path while its `Intent` changes a named existing test, fake or helper is a finding.
The finding names the member glyphs to list.
The existing sentence "only Go file paths of non-test files while its `Intent` names specific symbols" stays as it is, because a `wordingClaims` row pins it.
The rule is enforced by this stencil and this rubric only, with no mechanical plan-gate check.

Add `must` claims to the `loom-template-plan.md` and `loom-rubric-plan-review.md` rows of `wordingClaims` for the new wording.
The template row pins the test-member glyph rule, the `plan:` handle for a new test function and the `**Create:**` path for a new test file.
The rubric row pins the test-file finding.
Both `.md` files keep semantic line breaks, one sentence per line.

**ImpactSummary:** prose-only change to two loom stencils plus their claim rows in `contracts/stencils`, so no Go behavior changes; every later Plan-Write and Plan-Review round reads the new rule.
