# Card 3 — plan-stencil-test-rules

**Edit:**
- `contracts/stencils/loom/loom-template-plan.md`
- `contracts/stencils#wordingClaims`
- `contracts/stencils/loom/loom-rubric-plan-review.md`

**Intent:** Bring the plan writer's stencil and the Plan-Review rubric in line with `edit-test-file` (D1, D2, D5, and item 3 of the record's parent decision "Gate ways forward and Plan-Review rubric follow the test-file refusal").

In `contracts/stencils/loom/loom-template-plan.md`'s "Each `NN-<card-slug>.md`" section:
- The rule "An implementation card lists the implementation's member glyphs under `**Edit:**`, beside the existing test file that covers its surface." no longer names a test file: an implementation card lists the implementation's member glyphs under `**Edit:**`, beside the test members it changes or the `**Create:**` handles of the tests it adds.
- The rule "A `**Create:**` test file, after the implementation's `**Edit:**`, appears only for a surface no existing test file covers." no longer presumes a test file under `**Edit:**`: a new test file is a `**Create:**` path, written only for a surface no existing test file covers.
- The `**Edit:**` rule drops "a test file the card adds tests to" from the files with no symbol to name, leaving a non-Go file and a whole file the card creates or deletes, and drops "A plain test-file path stays only for a test file the card adds tests to without changing an existing member."; it states that a `_test.go` path under `**Edit:**` is refused by `edit-test-file` in `{{.specs_dir}}/loom/loom-plan-spec.md`, pointing at that row for the way forward rather than restating it.
- A new test function stays a `**Create:**` `plan:` handle, and the stencil says its unit is the package directory, or `<dir>_test` for an external test package, never a file, so the plan does not pin the file a new test goes in (D1).
- D2's rule is added: a card whose new tests reuse an existing test's setup, or call existing test helpers or fakes, names those members as glyphs under `**Uses:**`.
- The minimal skeleton's `- \`path/to/list_test.go\`` `**Edit:**` bullet is replaced by a test member glyph, `- \`internal/boardcli#TestList\``.

`contracts/stencils#wordingClaims`'s "the minimal skeleton shows member glyphs and an arrow head under Edit" claim swaps `- \`path/to/list_test.go\`` for the new skeleton bullet, and a new `wantAll` claim pins D2's `**Uses:**` rule and the `edit-test-file` pointer; a `wantNone` claim pins that "A plain test-file path stays only", "beside the existing test file that covers its surface", "after the implementation's `**Edit:**`, appears only" and "a test file the card adds tests to" are gone.

In `contracts/stencils/loom/loom-rubric-plan-review.md`'s "Member glyphs on an `Edit` card" bullet, the sentence "An `Edit` card that lists only a test-file path while its `Intent` changes a named existing test, fake or helper is a finding; the finding names the member glyphs to list." is dropped, since the gate now refuses that shape and the rubric's "Do not flag" section bars flagging what the gate checks.
`contracts/stencils#wordingClaims`' claim on the rubric that requires "lists only a test-file path while its `Intent` changes a named existing test, fake or helper" becomes a `mustNot` on the same text.

Webster's fork stencils are unchanged (D6).

**ImpactSummary:** The plan writer's stencil, one rubric sentence and their wording claims only; the deployed copies are reseeded by the normal stencil seeding, and no Go code reads the changed sentences.

**Commit:** `3: Plan stencil names new tests by handle and neighbours under Uses`
