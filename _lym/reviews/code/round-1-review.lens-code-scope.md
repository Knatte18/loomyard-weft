```yaml
lens: code-scope
outcome: reported
```

Restating the task: the code-scope lens traces each change in the diff to the board entry's goal points and the record's Added entry.

**Decisions (board entry `## Goal` points, plus the Added entry), with the changes that build each**

- **G1a** Merge danger section: Door `one-way`/`two-way`, Blast radius in one word. Built by C1.
- **G1b** The darn stencil asks for it. Built by C2.
- **G1c** The Describe stencil asks for it. Built by C3.
- **G1d** `lym loom validate-description` refuses a description without it. Built by C4.
- **G1e** A Code-Review rubric line checks the door against the diff. Built by C7.
- **G2a** A Test economy bullet for tautological tests, naming the test and the independent value. Built by C8.
- **G2b** `scribe:testing` gains the matching rule. A1 supersedes this, and no change is required.
- **G2c** A mutation survivor is evidence for the tautological finding. Built by C8.
- **G3a** One glossary file, one short line per term with the words never to use. It covers entry, decision record, recipe, Sketch and run. Built by C9 and C12.
- **G3b** The orch gets it through a heddle entry aimed at `orch-prompt`, never through `claude-prompt` or `agents-file`. Built by C9.
- **G3c** Discussion gets it in its stencil, calls out a term the entry uses against it, and a settled term goes into the glossary. Built by C10. Discussion records the glossary line as a decision rather than writing the file, since its write surface is its three files.
- **G3d** Writers, fixers and reviewers get nothing. Met by C9 and C10:
  - `orch-prompt` reaches the orch role only (`internal/heddleengine/target.go:88`).
  - The marker sits only in `loom-template-discussion.md`.
  - The advisor stencil, `gatefix-body-discussion.md` and the review rubrics do not include it (`contracts/stencils/loom/loom-template-discussion-advisor.md:21`, `contracts/stencils/rubric_test.go:52` names the writer path only).
- **G3e** A `ly:vocabulary` skill is optional ("may"). Not built, and that is allowed.
- **G3f** No lint. None is built, as required.
- **G3g** The `nicknames` entry stays; whether the terms ship in the binary was left to Discussion. The nicknames entry is untouched, and C9 ships the glossary in the binary.
- **A1** The `scribe:testing` rule is left out. No hunk touches scribe; the diff covers the darn and Describe stencils, the validator, the rubric and the glossary.

**Changes, with the decision each traces to**

- **C1** `contracts/specs/loom-final-summary-spec.md`, the Merge danger section: G1a.
- **C2** `contracts/stencils/darn/darn-template-write.md`, the close-with line: G1b.
- **C3** `contracts/stencils/landing/landing-template-describe.md`, the last list item: G1c.
- **C4** `internal/summaryparser/description.go` (checks, constants, `mergeDangerFindings`, `mergeDangerSection`, `isSectionHeading`, `labelledValue`) and `description_test.go` rows: G1d.
- **C5** Beam description surfaces, forced by C4: `internal/landingshed/beamdescribe.go` (`beamMergeDangerDraft`, the `beamDraft` tail), `beamdescribe_test.go`, `contracts/orchrole/role-beams.md`, and `internal/beamcli/deps_integration_test.go`.
  - `ValidateDescription` backs `RecordBeamDescription` (`beamdescribe.go:140`) and the pawl ready check (`beamcli/deps.go:460`), so a beam description without the section would now be refused.
  - The draft and the orch's beam rule have to carry the section for that path to keep working.
- **C6** Fixtures that stood for a valid description, forced by C4: `landingshed/describe_test.go`, `loomcli/parity_test.go`, `loomcli/validate_test.go`, `loomrecipe/fixture_test.go`.
- **C7** `loom-rubric-code-review.md`, the Merge danger bullet: G1e.
- **C8** `loom-rubric-code-review.md`, the tautological-test bullet in Test economy and the survivor-evidence sentence in the mutation section: G2a and G2c.
- **C9** `internal/heddleengine/glossary.md`, `shipped.go` (`Glossary`, `withGlossary`, `GlossarySlug`), `doc.go`, `store_test.go` row and `render_test.go` expectations: G3a and G3b. The doc and test edits are forced by the new shipped entry.
- **C10** `internal/loomengine/discussiontable.go` (the `glossary` marker and doc line) and `loom-template-discussion.md` (header note and "lym's words" section): G3c.
- **C11** `pattern/PATTERN-heddle-instructions.md` background line, forced by C9: the background describes what the shipped layer holds.
- **C12** `docs/reference/weaving-glossary.md`, a one-line pointer to the new glossary: G3a. The entry names this file as the one that "does not define what lym's terms mean", so the pointer closes the gap the entry describes.

**Findings:** none. Every change traces to a goal point or is forced by a traced change, and every goal point the Added entry leaves in scope is built.

Tool calls:
1. `sed -n 30,75p contracts/stencils/rubric_test.go` and a grep for `{{template` and `{{.glossary}}` in the Discussion stencils. My context held only the one `rubric_test.go:52` row and no view of which stencils include the Discussion template, which G3d ("reviewers get nothing") needs.
