---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/09-doc-go-check.md (Intent, the test-code rule)
    summary: docGoFindings holds a card whose only target in a doc.go package is a non-Go file such as template.yaml, where D8 holds only cards touching non-test Go code
    fix: In card 9's Intent, after the sentence that maps each ref to its package directory, add that a file target whose path does not end in .go is not Go code and is skipped, and in the Tests paragraph add a D8 row to TestValidatePlan_CardChecks where a card editing only a non-Go file internal/d/template.yaml in the doc.go package passes; state the same rule in the spec text card 9 adds for doc-go-unlisted.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:way-forward
    location: _lym/plan/07-test-tier-check.md (Intent, the pass rule and the finding text)
    summary: test-tier-unstated's only way forward is to add an Intent line, which cannot clear the two failing states the card pins, two tags without || and untagged beside a tag
    fix: In card 7's Intent, make the finding detail name the tags the file's naming lines hold, and extend the way forward after the add-a-line clause with "or, where the lines name several tags, join them with || for a helper several tiers share, else keep one; drop untagged or the tag when both appear"; have the tier0 tier1 and untagged-with-tier0 rows of TestValidatePlan_CardChecks also assert that clause in the Detail.
  - id: F3
    severity: MEDIUM
    class: scope
    origin: handler
    location: _lym/plan/08-seam-test-check.md and _lym/plan/09-doc-go-check.md (Tests paragraphs)
    summary: cards 8 and 9 repeat card 7's per-mode rows, done-card row and ValidateDispatch row for behaviour card 7's rows already pin, since every mode reaches planContextFindings through the same pc and pending filter
    fix: In card 9, drop the Edit targets internal/loomcli#TestGateParity_PlanGate, internal/loomcli#TestValidatePlanCmd, internal/loomcli#TestValidatePlanCmd_Overlay and internal/loomcli#TestValidatePlanCmd_ReworkReportsFindingsForFirstCardMismatch and their Tests sentences, since doc-go-unlisted adds no PlanContext field and card 7's rows in those tests already prove every mode reaches planContextFindings; in card 8, drop the Edit targets internal/loomcli#TestValidatePlanCmd, internal/loomcli#TestValidatePlanCmd_Overlay and internal/loomcli#TestValidatePlanCmd_ReworkReportsFindingsForFirstCardMismatch and their sentence, keeping the TestGateParity_PlanGate case and the TestPlanWriteEntry_GateConfig row, which pin that both builders fill SeamTests; in both cards drop the done-card and ValidateDispatch rows of TestValidatePlan_CardChecks, which card 7's rows cover, and keep each card's language csharp row, since each check has its own GoToolchain guard.
  - id: F4
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/04-citation-check.md (Create group and Tests paragraph)
    summary: card 4 creates the new file internal/loomshed/citations_integration_test.go through two member handles, where the Plan writer stencil makes a new test file a Create path, and D7's own check would not see the file
    fix: In card 4's Create group, replace the two handles plan:internal/loomshed#TestValidateLoomDiscussion_Citations and plan:internal/loomshed#unavailableIndex with the file target internal/loomshed/citations_integration_test.go, and keep the Tests paragraph naming the test, the unavailableIndex fake and the //go:build tier1 line; validate-plan --overlay accepts this with only the informational create-new-unit.
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/08-seam-test-check.md (Tests, the internal/c_test#TestDeps row)
    summary: the row listing internal/c_test#TestDeps under Uses while TestDeps sits in package c names a glyph quarry answers not_found, so the plan gate's resolve pass refuses that plan with glyph-not-found and the row passes only because it counts seam-test-unlisted alone
    fix: In card 8's Tests paragraph, keep the internal/c_test#TestDeps row and have it also assert that ValidatePlan reports glyph-not-found for that ref, so the row pins only the counterpart match and not a plan the gate passes; or replace the row with one where the marked test sits in an external-package file internal/c/c_ext_test.go (package c_test) and the card lists the resolving glyph internal/c_test#TestDeps under Uses, asserting no finding at all.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:way-forward
    location: _lym/plan/08-seam-test-check.md (Intent, the reader-error paragraph)
    summary: a seam-marker scan error halts the plan gate and names as its next step a session that may edit tests, but neither the plan writer nor the plan fixer may edit a test file, so the halt names no actor
    fix: In card 8's Intent, rewrite the reader-error sentence to name the actor and step, such as "the halt is the parent's, and the operator moves or corrects the marker in the task worktree as card 5's way forward says and re-runs lym loom validate-plan", and put the same sentence in the spec text card 8 adds.
  - id: F7
    severity: LOW
    class: design
    origin: lens:way-forward
    location: _lym/plan/06-seam-marker-scan.md (Intent, the seamViolations bullets)
    summary: a test calling NumField on a struct declared outside the module is refused unmarked, and refused again once marked because no worktree directory declares the struct, so neither way forward can succeed
    fix: In card 6's Intent, add a bound beside the non-test-helper bound saying a test reflecting directly over a struct from outside the module is not refused (the scan skips a NumField or VisibleFields call whose receiver's type it cannot place in the module), or give that case its own way forward; state the same bound in the PATTERN-test-speed background text card 5 writes.
  - id: F8
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/04-citation-check.md (Intent, the Path bullet)
    summary: card 4 reads the tracked root entries from HEAD's tree, which narrows D3's "an entry git tracks" to committed entries without a record entry licensing it
    fix: In card 4's Path bullet, read the root entries from the index git tracks (the entries staged or committed) so a staged root entry is tested, and drop the sentence naming the HEAD-tree narrowing as a bound; if gitrepo offers no index read, keep the HEAD read and record the narrowing through the parent as an Added after Discussion entry.
  - id: F9
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/04-citation-check.md (Intent, the Qualified symbol bullet)
    summary: card 4 strips only an empty trailing () where D4 ignores a trailing call parenthesis, so a citation such as gitrepo.New(worktreeRoot) is resolved as a member New(worktreeRoot) and refused as citation-symbol-missing
    fix: In card 4's Qualified symbol bullet, remove a trailing parenthesised argument list of any content, not only (), before matching pkg.Member, and add a row to TestValidateLoomDiscussion_Citations where real.Do(x) passes.
  - id: F10
    severity: NIT
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/03-index-resolve-glyphs.md (Edit group)
    summary: resolveTargets' doc comment lists every consumer of a batched Resolve answer, and card 3 adds ResolveGlyphs as a new consumer without touching that comment
    fix: In card 3, add internal/planglyph#resolveTargets under Edit and an Intent sentence that its doc comment names the consumers as a group, such as "every consumer of a batched Resolve answer", instead of listing them.
  - id: F11
    severity: NIT
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/04-citation-check.md (Edit group)
    summary: wireLightweight's doc comment says the validate verbs read only four path fields, and card 4 makes validate-discussion also read PlanIndex
    fix: In card 4, add internal/loomcli#loomCLI.wireLightweight under Edit and an Intent sentence that its doc comment names PlanIndex among the fields the validate verbs read.
  - id: F12
    severity: NIT
    class: design
    origin: lens:guard-reach
    location: _lym/plan/03-index-resolve-glyphs.md (Intent and TestResolveGlyphs rows)
    summary: ResolveGlyphs' empty-list branch and its test row cover a state no caller reaches, since cards 4, 8 and 9 each call it only with at least one glyph
    fix: In card 3, drop "an empty glyph list returns nil and opens no repository" from the Intent and the "an empty list returns nil" row from TestResolveGlyphs.
---

### [MEDIUM] Card 9 holds non-Go file targets to the package's `doc.go`

**Location:** `_lym/plan/09-doc-go-check.md`, Intent, the test-code rule.

**Issue:** `docGoFindings` maps every ref to its package directory and calls every ref that is not test code "non-test code".
So a card whose only target is `internal/gateslot/template.yaml`, in a package that holds `internal/gateslot/doc.go`, gets `doc-go-unlisted`.
D8 holds a card only when its targets "touch non-test Go code" (`_lym/discussion/decision-record.md:131`), and the acceptance criterion speaks of "production code".
I found such non-Go files in `doc.go` packages in the tree, so this refusal would fire on real plans.

### [MEDIUM] `test-tier-unstated`'s way forward cannot clear two of its own failing states

**Location:** `_lym/plan/07-test-tier-check.md`, Intent, the pass rule and the finding text.

**Issue:** The check counts the tags across every Intent line that names the file.
`tier0 tier1` without `||` and `untagged` beside `tier0` both fail, and the card's rows pin both failures.
The only way forward is "add an Intent line naming the file and its `//go:build` line".
Adding a line keeps the conflicting tags, so the re-prompted writer's fix still fails.
`PATTERN-refusal-way-forward` asks for a way forward that reaches a pass.
The wording D7 fixes is kept, and the fix only extends it.

### [MEDIUM] Cards 8 and 9 repeat mode and skip rows card 7 already pins

**Location:** `_lym/plan/08-seam-test-check.md` and `_lym/plan/09-doc-go-check.md`, Tests paragraphs.

**Issue:** Every mode reaches the new checks through `planContextFindings`, called by `ValidatePlan` and `ValidateReworkPlan` (`internal/loomshed/gates.go:269`, `:330`) with the `pc` each builder fills.
Card 7's rows in `TestGateParity_PlanGate`, `TestValidatePlanCmd`, `TestValidatePlanCmd_Overlay` and `TestValidatePlanCmd_ReworkReportsFindingsForFirstCardMismatch` already prove that every mode, and both builders, reach `planContextFindings`.
Card 9's `doc-go-unlisted` adds no `PlanContext` field, so its rows in those four tests pin nothing new.
Card 8's `SeamTests` field needs one pin per builder.
The `TestGateParity_PlanGate` case and the `TestPlanWriteEntry_GateConfig` row give that, which leaves its overlay, rework and `TestValidatePlanCmd` rows redundant.
The done-card row exercises `ValidatePlan`'s shared pending filter, and the `ValidateDispatch` row exercises an index method that never calls `planContextFindings`.
Card 7's rows already cover both.
The combined overlay with these target drops reports no new finding.

### [LOW] Card 4 creates a new test file through member handles

**Location:** `_lym/plan/04-citation-check.md`, Create group.

**Issue:** The Intent pins a new file, `internal/loomshed/citations_integration_test.go`, with a `//go:build tier1` line.
The Create group names it only through the handles `TestValidateLoomDiscussion_Citations` and `unavailableIndex`.
The Plan writer stencil says "A new test file is a `**Create:**` path" (`contracts/stencils/loom/loom-template-plan.md:176`).
Card 7's spec rule says the same, and D7's check reads only file targets.

### [LOW] Card 8's `_test` pseudo-unit row uses a spelling the plan gate refuses

**Location:** `_lym/plan/08-seam-test-check.md`, Tests, the `internal/c_test#TestDeps` row.

**Issue:** `lym quarry resolve 'internal/loomshed_test#TestNewPlanGate'` answers `not_found` (unit found) for a test declared in package `loomshed`.
So a plan listing `internal/c_test#TestDeps` under Uses for a test in package `c` fails the index's resolve pass with `glyph-not-found`.
The row reads as a pass only because the table counts `seam-test-unlisted` alone.
It pins a combination a real plan cannot reach.

### [LOW] The seam reader's halt names no one who can take its next step

**Location:** `_lym/plan/08-seam-test-check.md`, Intent, the reader-error paragraph.

**Issue:** A `*GuardScanError` from `SeamTestReader` is a returned error, so the plan gate halts.
The card names "a session that may edit tests" as the next step.
Neither the Plan-Write writer nor the Plan-Burler-Fix fixer may edit a test file, and the card names no role or command.

### [LOW] A reflective test over a struct outside the module has no admissible state

**Location:** `_lym/plan/06-seam-marker-scan.md`, Intent, the `seamViolations` bullets.

**Issue:** Unmarked, such a test is refused with "add a marker".
Marked, it is refused because no unit directory declares the struct.
No test does this today, but the scan would trap the first one that does.

### [LOW] Card 4 narrows D3's tracked root entries to HEAD's tree

**Location:** `_lym/plan/04-citation-check.md`, Intent, the Path bullet.

**Issue:** D3 tests a token whose first segment is "an entry git tracks at the worktree root".
The card reads HEAD's tree and states, as its own bound, that a staged root entry is not tested.
No decision-record entry licenses that narrowing.

### [LOW] Card 4 ignores only an empty trailing `()`

**Location:** `_lym/plan/04-citation-check.md`, Intent, the Qualified symbol bullet.

**Issue:** D4 says "A trailing call parenthesis is ignored".
The card removes only `()`.
So `gitrepo.New(worktreeRoot)`, a citation whose package and member exist, is resolved as `internal/gitrepo#New(worktreeRoot)` and refused as `citation-symbol-missing`.

### [NIT] `resolveTargets`' consumer list goes stale

**Location:** `_lym/plan/03-index-resolve-glyphs.md`, Edit group.

**Issue:** The doc comment above `resolveTargets` in `internal/planglyph/repo.go` lists every consumer of a batched Resolve answer.
`ResolveGlyphs` becomes one, and card 3 leaves that list untouched.

### [NIT] `wireLightweight`'s doc omits the index `validate-discussion` now reads

**Location:** `_lym/plan/04-citation-check.md`, Edit group.

**Issue:** `internal/loomcli/wiring.go:255-258` says the validate verbs read only four path fields.
After card 4, `validate-discussion` also reads `PlanIndex`.

### [NIT] `ResolveGlyphs`' empty-list branch is unreachable

**Location:** `_lym/plan/03-index-resolve-glyphs.md`, Intent and `TestResolveGlyphs`.

**Issue:** Cards 4, 8 and 9 each call `ResolveGlyphs` only with at least one glyph.
So the empty-list branch and its row defend a state no caller produces.

## Rejected

- Card 3's conditional on the `statusFindings` allowlist row is not a research step: `statusVocabularySelectors` holds `Unit`, and `statusFindings` keeps reading `r.Unit` for the not-found detail, so the row stays, as the card says.
- No card misses D10 or D11: card 4 carries the Discussion writer stencil sentence, which the chair gets through its `{{template "loom-template-discussion"}}` include; card 9 carries the Plan writer stencil sentence; cards 1, 4, 7, 8 and 9 carry the help texts; and card 5 carries `PATTERN.md` and `pattern/PATTERN-test-speed.md`.
- Card order holds: cards 8 and 9 use `ResolveGlyphs` (card 3), `goPackageDirs` (card 4), `FindSeamTests` (card 5) and `TestValidatePlan_CardChecks` (card 7), all from earlier cards, and no card uses `SeamTests` or `docGoFindings` before the card that adds it.
- The re-sign arrows are right: `ValidateLoomDiscussion`, `NewDiscussionGate` (card 4) and `planContextFindings` (card 8) carry them, and `ValidateDarnDiscussion`, `planContextOf` and `loomCLI.wire` keep their signatures.
- Card 6's marker list is complete: every `NumField`, `VisibleFields` and `NilSeams` call in a test file maps to a test the card marks, so no fact is left to find while it runs.
- No card carries a raw test command: no card has a `**Verify:**` line.

## Focus departures

- "Research steps: does any card leave a fact to be found while it runs, such as the exact set of reflective tests to mark in card 6, the `Env` nesting set, or which packages hold a `doc.go`?" — I settled the reflective-test set and the `doc.go` question from the tree. I did not enumerate the `Env` nesting set, because card 6 derives it with `envSeamStructs` and pins it against the real type in its own row, so it is the card's output, not a fact it depends on.
