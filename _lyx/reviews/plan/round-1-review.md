---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/01-master-read-set.md:18
    summary: Card 1's new opening paragraph tells the Master to read the repo's rules with no condition, so the empty-directive render names a read beyond the plan directory and a findings file, against D1's vacuous-when-empty wording and acceptance criterion 3
    fix: In card 1's opening-paragraph bullet (line 18), replace "you read the plan's overview and the repo's rules once, up front" with wording that names the rules only through the constraints section, for example "you read the plan's overview once, up front, plus the rules a constraints section below names, when that section is present"; keep the rest of the bullet. The prescribed sentence must not contain the string `pattern/`, so the card's count check stays equal.
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/00-overview.md:31
    summary: Card 1 edits tools/sandbox/SANDBOX-WEBSTER-SUITE.md, which tools/sandbox/suite.go embeds with go:embed, but the verify section neither vets nor tests ./tools/sandbox/...
    fix: In 00-overview.md's verify section, append ./tools/sandbox/... to each of the three go vet lines and both go test lines, after ./internal/websterengine/...; go build ./... stays as it is. Checked with lyx loom validate-plan --overlay against the {} baseline, and it reports ok.
---

### [MEDIUM] Opening paragraph names an unconditioned rules read

**Location:** `_lyx/plan/01-master-read-set.md:18`

**Issue:** D1 says "the wording names the rules through the constraints section and reads as vacuous when the pattern directive renders empty".
Acceptance criterion 3 says the empty-directive master render "names no read beyond the plan directory and a named gate findings file".
Card 1 holds the Orientation sentence and the `## What you never do` line to that rule, through the conditioned phrase.
The new opening paragraph, which sits above `{{.pattern_directive}}` in `webster-template-master.md` (lines 16 to 19 before the directives at lines 21 to 24), says "you read the plan's overview and the repo's rules once, up front" with no condition.
With PATTERN inactive, no constraints section renders, so this sentence tells the Master to read rules files nobody points it at.
The card's own pins do not catch it: the `pattern/` count check counts only the string `` `pattern/` ``, and the claim row's `wantNone` list does not cover "the repo's rules".

### [MEDIUM] Verify section skips the tools/sandbox package

**Location:** `_lyx/plan/00-overview.md:31`

**Issue:** The plan template requires the `## verify:` section to cover every package any card targets.
Card 1 edits `tools/sandbox/SANDBOX-WEBSTER-SUITE.md`.
`tools/sandbox/suite.go:48` embeds that file (`//go:embed SANDBOX-WEBSTER-SUITE.md`), which makes it an input of package `tools/sandbox`.
That package's tests (`tools/sandbox/suite_test.go`) are neither vetted nor run.
`go build ./...` compiles the package but runs none of its tests.

## Focus verdicts

- Focus 1, `notContains` "codebase" on the empty-directive render: not a defect.
  `RenderMasterPrompt` fills a template whose leading banner `stencil.FillOptional` strips.
  The only body lines holding "codebase" are 17, 18 and 27 of `webster-template-master.md`, and card 1 rewrites all three.
  With an empty parent name and note path, the parent, edit and friction directives render, and none contains the word (`contracts/stencils/edit/edit-directive.md`, `contracts/stencils/parent/*.md`).
- Focus 2, the `pattern/` count check: not a defect.
  The card prescribes the conditioned phrase in the Orientation sentence and the `## What you never do` line, and bars any other unconditioned `pattern/` mention, so both counts render as 2.
  They can differ only if the implementer writes the card's paraphrase "the `pattern/` background files it points at" next to the required phrase, which the card's "must contain the exact phrase" sentence rules out.
  The row is reached by a named input: `renderMaster` with `testLayout`, whose PATTERN directive renders empty.
- Focus 3, `prePackFreshRead` against the plain fragment: they agree.
  The body carries `## The FRESH-READ rule\n\n{{.fresh_read}}\n\n## Prior-batch context` (`webster-body-implementer.md` lines 3 to 7), and the fragment fills the marker with its banner and trailing newline stripped.
  After card 2, the fragment holds the two statement lines followed by the "Before you edit anything, read — in THIS turn — …" line, which is the card's prescribed constant line for line, blank line included.
- Focus 4, needles across rows: not a defect.
  `wordingClaims` rows check one stencil's raw text each, so the plain, pack and pack-card rows' `wantNone` never sees the inline fragment, whose row stays unchanged.
  Card 2 adds no render `notContains` of "one Master read before forking you" or "Inherited context can be stale.".
  The changed multipart-row needle "Before you edit anything, read — in THIS turn —" occurs only in the plain fragment (grep over `contracts/stencils`).
  "inheriting Master's context" occurs only in the fork title, and "You inherit Master's context, which includes…" in the unchanged IMPLEMENTER section does not match it.
  "codebase orientation", "Orient to the codebase" and "exactly like an in-session fork already holds it from Master" occur only at spans cards 2 and 3 rewrite.
- Focus 5, fidelity: D1 to D6 each have a card, including the gate findings file, the rules-file exception, the verify-fix line and the three docs.
  The plan adds no new top-level test, and no card edits the inline fragment, `webster-body-implementer.md` or `tools/forkaudit`.
  D7 and D8 correctly have no card.
  The one gap is F1.
- Focus 6, verify: the sections run in the template's order.
  `tools/sandbox` is the one gap (F2).
  `docs/overview.md` is not part of any package, so the verify rule does not cover it.
- Focus 7, glyphs: not a defect.
  `lyx quarry resolve` finds `contracts/stencils#wordingClaims` (var), `internal/websterengine_test#TestRenderPrompts` (func), `prePackStepOne` and `prePackFreshRead` (const) and `writePrePackBody` (func).
  None of them changes signature, so no re-sign arrow is needed.
  `internal/websterengine/doc.go` is listed as a file path because the Intent changes the package comment, which no member glyph names.

## Rejected

- Card 4's render needle and card 2's three render needles repeat claim-row phrases.
  Each is an assertion added to an existing row, not a new test function, so the rubric's test-economy check does not flag it.
- Card 2 renames the nil-pack row and rewrites the `prePackFreshRead` comment.
  Once the constant's bytes change, the old name and comment would be false, so the rename sits inside the Scope's "tests that pin the changed wording".
