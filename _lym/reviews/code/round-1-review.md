---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/summaryparser/description.go:124
    summary: The Merge danger scan ignores code fences, so a fenced example earlier in the body is taken as the section and masks the real one
    fix: "In mergeDangerSection, track whether a line sits inside a ``` or ~~~ fence and skip fenced lines both when matching the ## Merge danger heading and when isSectionHeading looks for the section's end. Add rows to TestValidateDescription_Table: a body whose only Merge danger section is inside a fence (want no-merge-danger); a fenced valid example followed by a real section with door revolving (want merge-danger-door); and a real section holding a fenced block with a # comment line before its Door and Blast radius lines (want nil)."
    origin: lens:code-correctness
  - id: F2
    severity: LOW
    class: design
    location: internal/heddleengine/glossary.md:7
    summary: Several glossary lines carry implementation detail the entry asks the glossary to leave out
    fix: "In internal/heddleengine/glossary.md, drop the storage and runtime details from the term lines while keeping each meaning and its Never words: remove 'kept on the Bolt checkout' from board, the file name `decision-record.md` from decision record, the path `_lym/landing/summary.md` from change description, and 'on the hub's tmux server' from strand (for example 'one agent's pane, holding one session')."
    origin: assist:assist
  - id: F3
    severity: LOW
    class: consistency
    location: internal/beamcli/cli.go:232
    summary: The beam describe --draft help, the beamcli package doc and the draft envelope's way_forward omit the Merge danger placeholders that recording now refuses
    fix: "In internal/beamcli/cli.go's describe help (the --draft paragraph), add that the draft ends with a Merge danger section whose door and blast radius are placeholders to call before recording. Add the same sentence to the describe --draft paragraph of internal/beamcli/doc.go. In internal/beamcli/describe.go:47, extend way_forward to say to call the Merge danger door and blast radius as well."
    origin: lens:code-affected-set
  - id: F4
    severity: LOW
    class: design
    location: internal/summaryparser/description.go:127
    summary: Survivors at lines 127 and 131 show no row puts the Door and Blast radius lines directly under the heading, so a section start shifted by one line goes unseen
    fix: "Add a TestValidateDescription_Table row with body \"# Title\\n\\nbody\\n\\n## Merge danger\\n**Door:** two-way\\n**Blast radius:** docs\\n\", want nil."
    origin: lens:code-tests
  - id: F5
    severity: LOW
    class: design
    location: internal/summaryparser/description.go:139
    summary: Survivor at line 139 shows no row ends the section at a level-one heading, so the "# " clause of isSectionHeading can be deleted unseen
    fix: "Add a TestValidateDescription_Table row with body \"# Title\\n\\nbody\\n\\n## Merge danger\\n\\n# Other\\n\\n**Door:** two-way\\n**Blast radius:** docs\\n\", want CheckMergeDangerDoor and CheckMergeDangerBlastRadius."
    origin: lens:code-tests
  - id: F6
    severity: LOW
    class: design
    location: internal/summaryparser/description.go:152
    summary: Survivor at line 152 shows the final-period strip labelledValue's doc comment promises is unchecked, since the only punctuated value is a blast radius that stays one word either way
    fix: "In TestValidateDescription_Table's row 'merge danger as list items in backticks', change the door line to \"- **Door:** `One-way`.\" so the row, which wants nil, fails without the period strip."
    origin: lens:code-tests
  - id: F7
    severity: LOW
    class: design
    location: internal/landingshed/beamdescribe.go:30
    summary: No test records the unedited beam draft and sees it refused, the contract beamMergeDangerDraft's comment states
    fix: "Add a TestRecordBeamDescription row (internal/landingshed/beamdescribe_test.go) whose description file is the draft DraftBeamDescription writes, unedited, and assert RecordBeamDescription refuses it with the merge-danger-door and merge-danger-blast-radius findings and does not commit."
    origin: lens:code-tests
  - id: F8
    severity: LOW
    class: design
    location: internal/loomengine/discussiontable.go:184
    summary: No test checks that the Discussion prompt carries the glossary text; any non-empty value in the glossary marker passes
    fix: "In the existing DiscussionTable test that renders the lead's prompt (TestDiscussionTable or TestProducerSpecs_SkillsAndParentDirective in internal/loomengine), assert that the rendered prompt contains strings.TrimSpace(heddleengine.Glossary())."
    origin: assist:assist
  - id: F9
    severity: NIT
    class: consistency
    location: internal/heddleengine/store_test.go:191
    summary: The new ShippedEntriesArePinned row pins the glossary's heading words, a new assertion on static prose
    fix: "In TestStore/ShippedEntriesArePinned, check the lym-glossary entry's first line only for its level (strings.HasPrefix(heading, \"## \")) instead of comparing it to the literal \"## lym's glossary\"; keep the slug, target and order checks."
    origin: assist:assist
  - id: F10
    severity: NIT
    class: consistency
    location: contracts/stencils/loom/loom-template-discussion.md:122
    summary: The glossary's own level-two heading follows the stencil's level-two heading directly, leaving that section empty and putting its instructions under the glossary's heading
    fix: "In loom-template-discussion.md's '## lym's words' section, move the instruction paragraph ('Hold the entry's words ...' through 'so the change adds the line to the glossary.') above {{.glossary}}, reword 'the glossary above' to 'the glossary below', so the section's text comes before the inserted glossary."
    origin: handler
---

### [MEDIUM] A fenced example satisfies the Merge danger check

**Location:** internal/summaryparser/description.go:124

**Issue:** `mergeDangerSection` takes the first line matching `## Merge danger` anywhere in the body, and `isSectionHeading` ends the section on any `# ` or `## ` line, fences included.
A description that shows the section's format in a fenced block before its real section is validated against the example: a missing or wrong real door passes.
A `# comment` line in a fenced shell block inside the real section ends it early and refuses a valid call.
This run's own `_lym/landing/summary.md` already has the first shape: its fenced example (`stencils`) precedes its real section (`descriptions`), and `lym loom validate-description` reads the example.
The spec itself shows the section in a fence, so writers are likely to copy it.

### [LOW] Glossary lines carry implementation detail

**Location:** internal/heddleengine/glossary.md:7

**Issue:** The board entry asks for a glossary free of implementation detail, with explanations kept in docs.
The board, decision record, change description and strand lines name a checkout, file names, a path and the tmux server.

### [LOW] beamcli's draft help, doc and way forward miss the placeholders

**Location:** internal/beamcli/cli.go:232

**Issue:** The draft now ends with a Merge danger section whose placeholders `RecordBeamDescription` refuses.
The `--draft` help (`cli.go:232-234`) and the package doc (`doc.go:137-141`) still list title, brief and commits only.
The envelope's `way_forward` (`describe.go:47`) tells the orch to edit the end result and member pull requests, never to call the door and blast radius, so its next step is refused.

### [LOW] Labels directly under the heading are untested

**Location:** internal/summaryparser/description.go:127

**Issue:** Every merge-danger row has a blank line after the heading, so mutants at lines 127 and 131 that start the section one line late survive.

### [LOW] A level-one heading ending the section is untested

**Location:** internal/summaryparser/description.go:139

**Issue:** The only terminating heading in the table is `## Notes`; deleting the `# ` clause survives.

### [LOW] The door's final-period strip is untested

**Location:** internal/summaryparser/description.go:152

**Issue:** `labelledValue` promises to trim a final period, but the only punctuated value is `` `board`. `` as a blast radius, which counts as one word without the trim.

### [LOW] Refusal of the unedited draft is untested

**Location:** internal/landingshed/beamdescribe.go:30

**Issue:** `TestHubLandDeps_DraftThenRecordStampsAndCommits` replaces the placeholders before recording, and no row records the draft as written.
Placeholders that happened to validate would pass every test.

### [LOW] Discussion's glossary value is unasserted

**Location:** internal/loomengine/discussiontable.go:184

**Issue:** The required marker guarantees a non-empty value only; swapping `heddleengine.Glossary()` for any other text passes every test.

### [NIT] The shipped-entry row pins the glossary heading's words

**Location:** internal/heddleengine/store_test.go:191

**Issue:** The new row compares the glossary's first line to `## lym's glossary`, a new assertion on static prose; the heading level is what the table's comment says it protects.

### [NIT] The glossary's heading swallows the stencil's section

**Location:** contracts/stencils/loom/loom-template-discussion.md:122

**Issue:** `## lym's words` is followed directly by the glossary's `## lym's glossary`, so the stencil's section is empty and its call-out instructions read as part of the glossary's section.

## Rejected

- `render_test.go` expecting `strings.TrimRight(Glossary(), "\n")` as a tautological test: this round's rubric has no tautology bullet, the expectation checks delivery of the glossary's one source rather than a computed value, and `passagesOf` trims the newline anyway.
- The `shipped.go:58` survivor: the mutated `"\n"` in `TrimRight` cannot be observed, since `passagesOf` trims newlines from every passage.
- Assist F3's MEDIUM severity: merged into F5 at LOW, since it is one missing table row for a small branch.
- Scope: no finding. Every change traces to a goal point or is forced by one; scribe is untouched as the parent's added decision asks, and the `nicknames` entry stays.
- Loosened checks: none. Every changed test assertion was updated for the new section or the glossary passage, not widened.
