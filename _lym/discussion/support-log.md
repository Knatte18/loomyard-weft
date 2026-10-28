# Support log — validator-checks

## Interview

Autonomous run: no operator answered; every pick below is an auto-pick recorded in the question ledger.

### Turn 1 — Exploration

Read the board entry (no `Decision for validator-checks:` note exists), `review-setup-tuning` and `turn-batching`, the audit report's sections 3 and 4, and the archived Plan reviews of `gate-runs-once` behind the seam, tier and `doc.go` themes.
Read `internal/loomcli/validate.go`, `internal/loomshed/gates.go`, `internal/discussionparser/validate.go` and its doc, `internal/planglyph/doc.go`, `internal/planindex/planindex.go`, the plan spec's card fields, `PATTERN-test-speed`, `PATTERN-leaf-packages` and the reflective tests in `internal/loomcli/landingdeps_test.go` and `internal/testkit/envkit/envkit_test.go`.
Facts found: decision ids are a convention only; `discussionparser` is stdlib-only by its doc; `planglyph` owns every quarry call; `impactset` parses `//lym:guard`; the darn gate runs at handoff, after the code; `lym loom decision add` runs `discussionparser.Validate`.

### Turn 2 — Discussion checks

Q: What is a decision id? Recommended: pin `### D<n> — <title>` in the spec and refuse other H3s, since a record without ids would pass the acceptance check vacuously. Auto-picked (D1).
Q: Which tokens does the citation check test? Recommended: backticked path tokens under an existing root entry, glyphs, and `pkg.Member` with a unique package directory; bare names untested. Auto-picked (D3, D4).
Q: How does a design cite a path or symbol it creates? Recommended: a ` (new)` or ` (removed)` mark after the token. Auto-picked (D3).
Q: Darn too? Recommended: D1 and D2 in both, citations loom only. Auto-picked (D2, D4).

### Turn 3 — Plan checks

Q: How does the check know a struct's reflective seam tests? Recommended: a `//lym:seam` marker kept complete by a scan. Auto-picked (D5, D6).
Q: How does a card state a new test file's tier? Recommended: Intent lines naming the file name one tier tag, a `||` helper disjunction or `untagged`. Auto-picked (D7).
Q: What is "a card whose Intent changes a package's behaviour"? Recommended: a card touching the package's non-test Go code, answered by listing its `doc.go` under Edit, Extends or Uses. Auto-picked (D8).

### Turn 4 — Rules in stencils

`PATTERN-stencil-ownership` forbids restating a format contract in a stencil, which the entry's "writer stencils state the new rules" would do read literally; the rules go in the specs and the stencils point at them (D10, Constraints).

Audit figures behind the picks: the code-claim theme holds 17 and the coverage-gap theme 14 of the gating Discussion findings, and the test-seam, docs-sync and coverage themes 16 gating Plan findings (report section 3).

### Turn 5 — Lens review

All seven forks returned a report.

- interactions 1, bound-breaker 2 (`Uses` way forward hits `uses-later-target`; the claimed ordering edge is false): fixed; D6 and D8 accept a later card's Edit or Extends as coverage, Uses only with no later editor, and the edge claim is gone.
- interactions 2, walkthrough 1 and 2 (`redundant-package-target` refuses the listing beside a package self glyph): fixed; a package self glyph target covers the doc or test (D6, D8).
- interactions 3 (a card deleting or moving a package cannot list `doc.go` again): fixed; D8 accepts the doc in any of the card's own groups.
- interactions 4, leanness 7 (acceptance lines name missing things on purpose): fixed; `<placeholder>` rule in D3 and D10, and this record's own criteria reworded.
- interactions 5 (mark versus ambiguity order): fixed; a marked token gets neither finding (D3, D4).
- interactions 6, existing-paths 3, walkthrough 4 (host layer of D6 to D8; `planGatePass` also runs under `--require-approved` and gets no tiers): fixed; D9 pins them to loomshed's plan context checks, carried by `PlanContext`, and `--require-approved` does not run them; the planGatePass note is dropped.
- existing-paths 1, walkthrough 3, bound-breaker 8 (`//lym:seam` beside `//lym:guard` trips `GuardMisplaced`): fixed; D5 lets the three marker lines stack in any order, and both parsers admit the others.
- existing-paths 2, bound-breaker 4 (suffixed `## PATTERN changes` bullets are writer-uneditable): fixed; D3 and D4 skip them.
- bound-breaker 1 (headings and `C#` read as glyphs): fixed; the glyph rule needs a quarry-grammar parse, no whitespace or `<`, and a unit holding `/` or `.`.
- bound-breaker 3, walkthrough 6 (handles, declaring file and package self glyph miss D6): fixed; D6 is held by each of them.
- bound-breaker 5 (quarry answers beyond found and not found): fixed; D4 uses `glyph-not-found`'s status policy.
- bound-breaker 6 (new quarry dependency at Discussion): fixed; stated in D4's bound.
- bound-breaker 7 (same-package reflective helpers): fixed; the scan follows a same-package `_test.go` helper, and the acceptance line says "tests the scan sees".
- bound-breaker 9 (darn runs missing from the risk): fixed in Open risks.
- walkthrough 5 (D7 sees the canonical glyph, not the written path): fixed; D7 matches the glyph's unit path or base name.
- walkthrough 7 (`testdata` packages in the base-name map): fixed; go-ignored directories are left out (D4).
- goal-scope F1 (unqualified names narrow "every"): fixed; Scope says so.
- goal-scope F2 (darn exclusion unlicensed): fixed; listed under Auto-mode assumptions as a narrowing, rationale kept in D4.
- goal-scope F3 (marker wider than `Deps`/`Env`): fixed; D5 states the widening and why, and Auto-mode assumptions lists it.
- goal-scope F4 (D7's relaxations): fixed; D7's text names the entry's case and why the other two are allowed.
- board-fit 1 (open risk on `review-setup-tuning` already settled by its record): fixed; bullet dropped.
- board-fit 2 (`landing-settles-in-place` changes a reflective test): fixed; Open risks names it.
- leanness 1 (Auto-mode assumptions repeat decisions): fixed; only the readings of the entry remain.
- leanness 2 (notes bullet repeats D4): fixed; dropped, wiring fact folded into D4.
- leanness 3 (D1's spec sentence overlaps D10): fixed; dropped from D1.
- leanness 4 (D11 restates the docs rule and hedges): fixed; D11 keeps the help texts and PATTERN background.
- leanness 5 (Constraints restate `PATTERN-test-speed`): fixed; bullet cut.
- leanness 6 (audit tallies in rationale): fixed; moved to Turn 4 above.

## Rejected alternatives

- Citation check on unqualified names (`MessageLine`): hook names, JSON keys and constants share the shape, so false findings would be routine.
- Symbol resolution by `go/parser` inside the discussion check: a second resolver beside quarry, Go-only, drifting from the plan's.
- Exempting a missing path whose parent directory exists: lets a typo in an existing package through.
- No exemption for new paths: refuses every design that creates a file.
- Seam tests found by struct name (`*Deps`, `*Env`) or by grepping test text for the type: misses structs and tests, and flags non-seam reflection.
- A hand-kept registry of struct to seam tests: rots without a scan, and the marker plus scan is the same cost.
- A new card field for docs (`**Docs:**`): the plan format admits no `none` sentinel, and `Uses` already means "read, unchanged".
- Requiring a `doc.go` listing only when the doc names the edited member: misses the falsified-sentence findings the audit shows, which name behaviour, not members.
- Citation check on darn: tests the code at handoff, not the claim, and saves no review round.

## Review rounds

_No rounds yet._

## Question ledger

- Decision id form — resolved, auto-pick: `### D<n> — <title>` pinned (D1).
- Acceptance check reach — resolved, auto-pick: loom and darn, never `lym loom decision add` (D2).
- Citation token rules and marks — resolved, auto-pick (D3, D4).
- Citation check on darn — resolved, auto-pick: no (D4).
- Seam-test source — resolved, auto-pick: `//lym:seam` marker and scan (D5).
- Tier statement form — resolved, auto-pick (D7).
- `doc.go` proxy — resolved, auto-pick (D8).
- Stencil restating rules versus `PATTERN-stencil-ownership` — resolved by the PATTERN entry (D10).
