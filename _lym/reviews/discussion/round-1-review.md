---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D4, sentence on resolution through the code-index seam
    summary: D4 resolves citation glyphs through planindex.Index, but no Index method takes a bare glyph list; every method takes a planparser.Plan, and the glyph-not-found status policy is built per card
    fix: In D4, replace the resolution sentence with one stating that planindex.Index gains a method resolving a list of glyphs against a worktree root, implemented in internal/planglyph with the glyph-not-found status policy lifted out of its per-card form, so an ambiguous answer split by build constraint still resolves and a quarry failure still wraps planindex.ErrQuarryUnavailable.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D5, the scan sentence (calls NumField or VisibleFields on a reflect value)
    summary: D5's scan trigger names NumField or VisibleFields on a reflect value, but most of today's reflective tests call NumField on a reflect.Type and VisibleFields is the package function reflect.VisibleFields, so a literal scan misses them
    fix: In D5, reword the scan trigger to a test function that calls the NumField method on a reflect.Type or reflect.Value, or calls reflect.VisibleFields, directly or through a same-package _test.go helper or envkit.NilSeams; leave the rest of the sentence unchanged.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D8, first sentence (targets touch non-test Go code)
    summary: An internal-package test function's glyph shares its unit with production code (internal/loomshed#TestNewDiscussionGate), and D8 runs in planContextFindings with no index, so a test-only card editing tests by member glyph reads as touching non-test code and is refused, contradicting D8's bound and its acceptance line
    fix: In D8, state how a target is classified as test or non-test code - a file target by its _test.go suffix, a _test pseudo-unit as test, and a member glyph by the file quarry reports declaring it, resolved through the index the plan gate already holds - and that a member glyph declared only in _test.go files counts as test code.
  - id: F4
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md D6 Bound and D8 Bound
    summary: The Uses listing in D6 and D8 is an escape hatch that passes the check whatever the card does, yet neither Bound says so; D8's Bound names only the Extends and later-card holes, and D6's Bound names none for Uses
    fix: Add to D6's Bound - a seam test listed under Uses passes even when the card's change breaks it; the card gate's test run and review catch that. Add to D8's Bound - a doc.go listed under Uses passes even when the card falsifies its text; review catches that.
  - id: F5
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D9, first sentence (PlanContext carries the gate.yaml tier tags and the seam markers)
    summary: PlanContext is built twice, by validatePlanCmd and by shedrecipe's planContextOf, and D9 does not say what scans the worktree for seam markers or whether PlanContext holds a value scanned at wire time or a reader called at check time, so gate and verb can diverge or read stale markers
    fix: In D9, add that PlanContext carries the tier tags as a value read from gate.yaml, as RefuseRaw is, and the seam markers as a reader over the worktree root called at check time through the impactset marker parser, and that both PlanContext builders fill the two fields the same way.
  - id: F6
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D2, second paragraph
    summary: D2 puts D1 and D2 beside discussionparser.Validate in both callers but leaves their order unstated, while ValidateDarnDiscussion returns early on any Validate finding and ValidateLoomDiscussion runs every check so one re-prompt names every defect
    fix: In D2's second paragraph, add that both callers run D1 and D2 even when Validate finds defects and append their findings after Validate's, so one re-prompt names every defect; in the darn caller the history check keeps running only on a record Validate finds clean.
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D1, second paragraph
    summary: D1's decision-heading-malformed and decision-id-duplicate findings name no way forward, while the Constraints bullet on PATTERN-refusal-way-forward requires every new finding to name one
    fix: "In D1, add the ways forward - for decision-heading-malformed, rewrite the heading as ### D<n> — <title>; for decision-id-duplicate, renumber one of the decisions and the acceptance bullets that name it."
  - id: F8
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D3, second sentence (first segment is an entry at the worktree root)
    summary: D3 decides whether a token is tested by the worktree root on disk, so an untracked machine-local entry such as .scratch, .dev-bin or the worktree's .git file makes the same record pass on one machine and fail on another
    fix: In D3, change the root-entry rule to an entry git tracks at the worktree root, other than the _lym and .lym roots internal/lymdirs names, and add .git to the excluded first segments.
  - id: F9
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D3, second sentence (the path shape rule)
    summary: A Go package pattern such as internal/... has a slash, no whitespace and none of the excluded characters, so D3 tests it and raises a false citation-path-missing
    fix: In D3, add that a token holding a ... segment is not a path, beside the excluded characters.
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D5, the scan sentence (calls envkit.NilSeams)
    summary: Inside package envkit, NilSeams is called unqualified (envkit_test.go:17 over shedrecipe.Env) and lives in the non-test envkit.go, so neither the envkit.NilSeams arm nor the same-package _test.go helper arm sees that seam test
    fix: In D5, reword the NilSeams arm to a call of envkit.NilSeams, spelled NilSeams inside package envkit.
  - id: F11
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D7, last sentence of the first paragraph (tier tags are those gate.yaml declares)
    summary: gate.yaml keys its tiers tierN whatever build tag the tier command passes, so on a Go hub whose commands use other -tags values D7 would demand a tag the file must not carry
    fix: In D7, state that the tier tags are the gate.yaml tier keys, which on a Go hub are the build tags the tier commands pass with -tags, as PATTERN-test-speed's tierN rule requires; no other change.
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D10, third sentence (Discussion writer and chair stencils each gain one sentence)
    summary: The chair stencil includes loom-template-discussion whole, so adding the pointer sentence to both renders it twice in the chair's prompt
    fix: In D10, say the Discussion writer stencil and the Plan writer stencil each gain the pointer sentence, and that the chair stencil gets it through its include of the writer stencil; update D10's acceptance bullet to match.
  - id: F13
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md Acceptance criteria, the D5 bullet
    summary: D5 also refuses a marker whose glyph does not name a struct, and names reflect.VisibleFields and envkit.NilSeams as triggers, but the D5 acceptance bullet covers only NumField and the stacking order
    fix: Extend the D5 acceptance bullet with - a //lym:seam marker whose glyph names a non-struct fails the scan; a test calling envkit.NilSeams without a marker fails the scan.
  - id: F14
    severity: LOW
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md D3, last sentence of the first paragraph and the Bound paragraph
    summary: The <placeholder> rule exempts every token holding a < from the path and symbol tests, an escape hatch the Bound paragraph does not name beside the marks
    fix: In D3's Bound, add - a token holding a < is not tested at all, so a writer can exempt a path by adding a placeholder part; a reviewer still reads the claim.
  - id: F15
    severity: LOW
    class: scope
    origin: lens:goal-scope
    location: _lym/discussion/decision-record.md Auto-mode assumptions
    summary: D6 widens the entry's seam-test rule - it fires on any target naming the struct, a member, a handle, the declaring file or the package self glyph rather than on a field added, and accepts a later card's Edit or Extends as cover - and Auto-mode assumptions does not list either reading
    fix: Add one Auto-mode assumptions bullet - a card is held to a marked struct when any target names it, not only when it adds a field, and a later card that edits the test covers it (D6).
  - id: F16
    severity: NIT
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D10, first sentence
    summary: D10 says the discussion spec states D1 to D4 for loom and darn records, which reads as if the citation rules apply to darn records, while D4 runs D1 and D2 only on darn
    fix: In D10's first sentence, say the spec states D1 and D2 for loom and darn records and D3 and D4 for loom records.
---

Every focus question was settled against the code:

- **D2, D4, D5 and D9 code claims.**
  `shedrecipe.Env` holds `WorktreeRoot` (`internal/shedrecipe/recipe.go:88`) and `PlanIndex` (`recipe.go:279`), and the loom wiring fills both (`internal/loomcli/wiring.go:550,582`).
  `ValidateLoomDiscussion` and `ValidateDarnDiscussion` exist with the path-only shapes the record extends (`internal/loomshed/gates.go:123,139`).
  `impactset` parses `//lym:guard` (`internal/impactset/guards.go:24`), and today it admits only keep lines after the marker; D5 states that change.
  The one false premise is resolution through `planindex.Index` (F1).
- **`PATTERN-test-speed` bullet.**
  Its rule text is about 236 characters, within `pattern.MaxEntryTextChars` (280), and it uses the spec's `— changes:` form.
- **Attack surface.**
  The marks (D3, D4), the `untagged` word (D7, bounded by `cmd/lym/tiertags_test.go`), the darn/loom split (D4's bound and rationale) and the `lym loom decision add` carve-out (D2) state their bounds.
  The Uses listing (F4) and the `<placeholder>` rule (F14) do not.
- **Acceptance coverage.**
  Every id from D1 to D11 appears as a whole word in a bullet. The D5 bullet misses one stated refusal (F13).
- **Deferred research.**
  No Decision, Constraint or Open risk hands a fact downstream.
  The marker source left unstated in D9 is a design gap (F5), not a deferred fact.
- **Relocation.**
  `decision-record.md` carries no deliberation narrative. The audit tallies already sit in the support log's Turn 4.

### [MEDIUM] D4 resolves through an Index method that does not exist

**Location:** `_lym/discussion/decision-record.md` D4

**Issue:** Every `planindex.Index` method takes a `*planparser.Plan` (`internal/planindex/planindex.go:79-102`).
The `glyph-not-found` policy builds per-card findings (`internal/planglyph/resolve.go:84-118`).
D4 treats resolving a bare glyph list as something the seam already does.

### [MEDIUM] D5 scan trigger misses reflect.Type and reflect.VisibleFields

**Location:** `_lym/discussion/decision-record.md` D5

**Issue:** Most reflective tests call `typ.NumField()` on a `reflect.Type`:

- `internal/websterengine/config_test.go:49`
- `internal/fabricengine/mergevocab_test.go:205`
- `internal/fabricengine/mergestate_integration_test.go:42`
- `internal/loomengine/config_test.go:1117`

`VisibleFields` is a package function.
Read literally, "on a reflect value" leaves these tests out.

### [MEDIUM] D8 cannot tell a test member glyph from production code

**Location:** `_lym/discussion/decision-record.md` D8

**Issue:** `internal/loomshed#TestNewDiscussionGate` has the same unit as production members.
`planContextFindings` gets no index (`internal/loomshed/plancontext.go:55`), so D8 refuses a test-only card that edits tests by member glyph.
D8's own bound and its acceptance line say such a card passes.

### [MEDIUM] Uses listing in D6 and D8 is an unbounded escape hatch

**Location:** `_lym/discussion/decision-record.md` D6 and D8 Bounds

**Issue:** Listing the test or `doc.go` under Uses satisfies the check whatever the card changes.
The rubric's attack-surface rule requires the Decision to state what this lets through and what bounds it.

### [LOW] D9 leaves the seam-marker source and its two builders unstated

**Location:** `_lym/discussion/decision-record.md` D9

**Issue:** `PlanContext` is built in `internal/loomcli/validate.go:271-277` and in `internal/shedrecipe/entries_gate.go:126-133`.
Neither D5 nor D9 says what scans the markers, or when.

### [LOW] D2 check order against the darn early return is unstated

**Location:** `_lym/discussion/decision-record.md` D2

**Issue:** `ValidateDarnDiscussion` returns on any `Validate` finding (`internal/loomshed/gates.go:140-143`), while the loom check reports every defect in one pass.

### [LOW] D1 findings name no way forward

**Location:** `_lym/discussion/decision-record.md` D1

**Issue:** The record's own `PATTERN-refusal-way-forward` constraint requires a way forward in every new finding.

### [LOW] D3 root-entry test reads untracked entries

**Location:** `_lym/discussion/decision-record.md` D3

**Issue:** Whether a token is tested depends on local untracked root entries, so the result changes by machine.

### [LOW] D3 tests Go package patterns as paths

**Location:** `_lym/discussion/decision-record.md` D3

**Issue:** `internal/...` passes the path-shape rule and is reported missing.

### [LOW] D5 misses the unqualified NilSeams call inside envkit

**Location:** `_lym/discussion/decision-record.md` D5

**Issue:** `internal/testkit/envkit/envkit_test.go:17` calls `NilSeams(FullEnv(t))` unqualified, and `NilSeams` lives in a non-test file (`envkit.go:521`).

### [LOW] D7 equates gate.yaml tier keys with build tags

**Location:** `_lym/discussion/decision-record.md` D7

**Issue:** `internal/gateslot/config_test.go:116` shows key `tier1` with `-tags integration`.
The keys and the tags agree only by convention.

### [LOW] D10 pointer renders twice in the chair prompt

**Location:** `_lym/discussion/decision-record.md` D10

**Issue:** `loom-template-discussion-chair.md` includes `loom-template-discussion` whole (line 8).

### [LOW] D5 acceptance bullet omits the marker-glyph refusal

**Location:** `_lym/discussion/decision-record.md` Acceptance criteria

**Issue:** No scenario covers a marker whose glyph names a non-struct, or a test that calls `envkit.NilSeams`.

### [LOW] D3 placeholder exemption is not in its Bound

**Location:** `_lym/discussion/decision-record.md` D3

**Issue:** Any token holding `<` skips both tests, and the Bound names only the marks.

### [LOW] D6's widening of the entry is not an Auto-mode assumption

**Location:** `_lym/discussion/decision-record.md` Auto-mode assumptions

**Issue:** The entry fires on "a card that adds a field" and asks the card to list the tests.
D6 fires more widely and also accepts a later card as cover.

### [NIT] D10 reads as applying citation rules to darn

**Location:** `_lym/discussion/decision-record.md` D10

**Issue:** "D1 to D4 for loom and darn records" conflicts with D4's loom-only citation check.
