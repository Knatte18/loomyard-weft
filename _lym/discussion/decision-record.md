# Decision record — validator-checks

## Goal

`lym loom validate-discussion` and `lym loom validate-plan`, and the gates that call the same functions, refuse the recurring review findings a machine can see, before a reviewer spends a round on them.
`docs/research/review-setup-audit-2026-10-28/report.md` section 3 names the themes: wrong code claims and uncovered decisions in the decision record; missing seam tests, tier tags and `doc.go` files in plan cards.
Each finding names the item it found and what the writer adds to fix it, and the writers meet the rules through the specs their stencils point at.

## Scope

In:

- Discussion: a decision-id check, an acceptance check (D1, D2) and a citation check (D3, D4).
- Plan: a seam-test check (D5, D6), a tier-statement check (D7) and a `doc.go` check (D8), with where they run (D9).
- The specs, stencil pointers, CLI help and PATTERN line the checks change (D10, D11).

Out:

- Lens and profile changes, which `review-setup-tuning` builds.
- Checks for the design-interaction and unbounded-edge classes, which the report leaves to the lenses.
- The report's other machine-check ideas: every acceptance line maps to a card, and a refusal sentence names a command.
- Checking the relevant list's paths and members.
- Unqualified names such as `MessageLine` or `Stop`: this narrows the entry's "every Go symbol" to the shapes D4 can tell from hook names, JSON keys and constants.

## Decisions

### D1 — Writer decisions carry a `D<n>` id heading

Every writer-authored H3 under `## Decisions` is `### D<n> — <title>`, ids unique within the record.
An H3 there that is neither such a heading nor an `### Added after Discussion` entry is a `decision-heading-malformed` finding, with the way forward: rewrite the heading as `### D<n> — <title>` when it is a decision, or demote it to H4 or plain text when it is not.
A repeated id is a `decision-id-duplicate` finding, with the way forward: renumber one of the decisions and every mention of its id.

Rationale: the acceptance check needs a machine-readable id; today's records use `D<n>` by convention only, and a record without ids would pass D2 vacuously.

### D2 — Every decision id appears in an acceptance criterion

Each `D<n>` id from D1's headings appears, as a whole word, in at least one bullet under `## Acceptance criteria`; otherwise a `decision-not-in-acceptance` finding names the id, with the way forward: name it in the criterion that covers it, or add one.
A range such as "D1 to D4" counts only the ids it spells.
`Added after Discussion` entries carry no id and are not checked.

D1 and D2 run in one function beside `discussionparser.Validate`, never inside it, called by the loom check `loomshed.ValidateLoomDiscussion` and the darn check `loomshed.ValidateDarnDiscussion`, so both recipes' gates and `validate-discussion` run them.
Both callers run D1 and D2 even when `Validate` finds defects and append their findings after `Validate`'s, so one re-prompt names every defect;
they add nothing on a record `Validate` reports missing, and D2 adds nothing on a record missing `## Acceptance criteria`, whose `section-missing` finding already names the defect;
in the darn caller the history check keeps running only on a record `Validate` finds clean.
`lym loom decision add` keeps running `Validate` alone, so a decision added to a run whose record predates D1 is never refused for the record's older shape.

Rationale: the coverage-gap theme is pure text, and the darn record has the same required section.

### D3 — Citation check: cited paths exist

On a loom run, every backticked token in the decision record that reads as a repository path names a file or directory in the worktree.
A token reads as a path when it holds a `/`, no whitespace, none of `#`, `<`, `>`, `*`, `{`, `}`, `$`, `:` and no `...` segment, and its first segment is an entry git tracks at the worktree root, other than the `_lym` and `.lym` roots `internal/lymdirs` names; a trailing `/` names a directory.
A token that fails is a `citation-path-missing` finding naming the token and its line, with the way forward: correct the path, or mark one the task creates or removes with ` (new)` or ` (removed)` right after the closing backtick.
A name that must not exist, as in a negative acceptance scenario, is written with a `<placeholder>` part, which no rule here tests.

Bound: a marked token is not tested at all, so a writer can exempt a wrong token by marking it; a reviewer still reads the claim.
A token holding a `<` is not tested at all either, so a writer can exempt a path by adding a placeholder part; a reviewer still reads the claim.
A path outside backticks, a slash-free file name, a Go package pattern such as `internal/...`, a token under a root entry git does not track (a Bolt path, an import path, `.scratch/`) and a run-state path are not tested.

### D4 — Citation check: cited symbols exist

On a loom run, two token shapes are symbol citations and must resolve:

- a glyph: a token with no whitespace and no `<`, not `plan:`-prefixed, that parses under quarry's glyph grammar as `<unit>#<member>` with a unit holding a `/` or a `.` and naming no existing worktree file except a `.go` or `.cs` file, the extensions quarry has an alphabet for, so a backticked heading (`## Decisions`), `C#`, `#123` or a markdown anchor such as `docs/overview.md#Modules` is not one;
- `pkg.Member` or `pkg.Member.Sub`, a token with no whitespace and no `<`, where `pkg` is the base name of exactly one Go package directory of the worktree, outside the directories the go tool ignores (`testdata`, `_`- and `.`-prefixed), and `Member` holds an upper-case letter; it is resolved by its `pkg.Member` prefix alone, as the glyph `<dir>#Member`, since quarry gives a Go struct field no glyph. A trailing call parenthesis is ignored.

When `pkg` names two or more package directories, a `citation-symbol-ambiguous` finding names them, with the way forward: spell the symbol as a glyph.
A token that does not resolve is a `citation-symbol-missing` finding with the way forward: correct it as `lym quarry glyphs <dir>` lists the members, or mark it ` (new)` or ` (removed)` as D3 says.
A marked token gets neither finding.
`planindex.Index` gains a method resolving a list of glyphs against a worktree root, implemented in `internal/planglyph` with the `glyph-not-found` status policy lifted out of its per-card form, so an ambiguous answer split by build constraint still resolves.
An ambiguous answer not split by build constraint is a `citation-symbol-ambiguous` finding naming the candidates' files, with the way forward: cite the declaring file meant by its path, which D3 tests, in place of the symbol.
An answer quarry rejects before resolution is a `citation-symbol-missing` finding with the way forward above.
The method also reports the files declaring each resolved glyph, which D6 and D8 read.
A quarry outage is a returned error wrapping `planindex.ErrQuarryUnavailable`, never a pass and never a finding.

Bound: a lower-case member (`state.json`), a package outside the worktree (`os.ReadFile`) and an unqualified name are not tested; the spec tells the writer to qualify a Go symbol with its package.
A field or method after the first member of a `pkg.Member.Sub` token is not tested.
The Discussion-Write and Discussion-Burler-Fix gates now depend on quarry: an outage halts them as an infrastructure fault, where today they pass.

D3 and D4 skip fenced code blocks, `### Added after Discussion` entries, `## PATTERN changes` bullets carrying the ` — added after Discussion` suffix and `<!-- lym:carry-over … -->` blocks, which no writer may edit.
`internal/discussionparser`, the record's sole parser, extracts the citations with their line and mark; `internal/loomshed` tests them against the told worktree root and index, so `discussionparser` stays stdlib-only.
`ValidateLoomDiscussion` gains the worktree root and index; the gate gets them from `shedrecipe.Env`, which already holds both, and the verb from its loom wiring.

Rationale: the code-claim theme is mostly a path or symbol that is not there.
The darn record is checked at handoff, after the code, so existence would test the code and not the claim, and darn has no Discussion review round to save; the darn check runs D1 and D2 only.

### D5 — `//lym:seam` marks a test that reflects over a struct's fields

A test function whose body reflects over a struct's field set carries `//lym:seam <unit>#<Type>`, one line per struct, in the comment block directly above it.
`//lym:seam`, `//lym:guard` and `//testtiming:keep` lines stack there in any order: the guard scan admits seam lines between `//lym:guard` and the func, and the seam parser admits guard and keep lines.
The marker is parsed beside `//lym:guard` in `internal/impactset`.
The marker covers every struct a test reflects over, config and record structs included, not only `Deps` and `Env`, since adding a field to any of them breaks the same way.
A test's markers name every struct its reflection walks, nested struct fields included: a test reaching `envkit.NilSeams` marks `internal/shedrecipe#Env` and each struct type `Env` nests by value, recursively, through an exported field whose path `envkit`'s `nilLegal` set does not skip, as `NilSeams` walks.
This task marks today's tests, and a tier0 scan refuses, per package, a test function without the marker that calls the `NumField` method on a `reflect.Type` or `reflect.Value`, or calls `reflect.VisibleFields`, directly or through a same-package `_test.go` helper or a call of `envkit.NilSeams`, spelled `NilSeams` inside package `envkit`;
it also refuses a marker whose glyph does not name a struct, and a test reaching `NilSeams` whose markers miss a struct of the nested set the scan derives from the `Env` type.
Each refusal names the test and its way forward: add a `//lym:seam <unit>#<Type>` line for each struct it reflects over or the scan names as missing, or correct a marker's glyph to name the struct the test reflects over.

Bound: reflection reached through any non-test helper, in the same package or another, is not seen, except `envkit.NilSeams`, which the scan names.
For a test reflecting directly, the scan checks that a marker exists and names a struct, not that the markers name every struct the test walks; review checks that.

Rationale: the plan check needs to know which tests are a struct's seam tests; the scan guarantees every reflective test it sees is marked, and for `NilSeams` that the markers cover every struct it walks, where guessing from the type's name or the test's text guarantees neither.

### D6 — Seam-test check in the plan

A card is held to a marked struct when its Create, Edit, Extends, Delete or Rename targets name the struct, a member of it, a `plan:` handle whose unit and member name one (the `plan:` prefix stripped), the file that declares the struct, or its package self glyph.
Such a card covers each test marked for that struct when one of these holds:

- its own groups name the test by glyph, name its file under Uses, Delete, Rename or Move, or name the package self glyph of the test's package or its `_test` pseudo-unit;
- a later card names the test under Edit or Extends.

Otherwise a `seam-test-unlisted` finding names the card, the struct and the test, with the way forward: list it under Edit by its member glyph when the card changes it, under Uses when it stays as is and no later card targets it, else leave it to the later card that edits it.
A test glyph matches across `<dir>` and its `<dir>_test` pseudo-unit, as `planparser.GoPackageCounterpart` maps them.

Bound: a method change on a marked struct also requires the listing; a field added through a target D6 does not name above is not seen.
A seam test listed under Uses passes even when the card's change breaks it; the card gate's test run and review catch that.
A seam test a later card names under Edit or Extends passes the earlier card even when the later card edits it for another reason, and the test stays broken between the two cards; the earlier card's gate test run and review catch that.

### D7 — Tier-statement check in the plan

For each `_test.go` file a card's Create group names as a file target, the card's Intent lines that name the file, by the canonical glyph's unit path or by its base name, state its constraint.
Together they name exactly one tier tag, the entry's case; or several only joined by `||`, for a helper several tiers share; or the word `untagged` and no tier tag, for a support-only file, since `PATTERN-test-speed` allows both.
Otherwise a `test-tier-unstated` finding names the card and file, with the way forward: add an Intent line naming the file and its `//go:build` line, such as `//go:build tier0`.
The tier tags are the hub's `gate.yaml` tier keys, which on a Go hub are the build tags the tier commands pass with `-tags`, as `PATTERN-test-speed`'s `tierN` rule requires; with no `gate.yaml` the check is skipped.

Bound: a test file created as a member glyph and not as a file target is not seen; the plan spec tells the writer to name a new test file as a Create file target.
Whether the stated tier fits the test stays with `cmd/lym/tiertags_test.go` at the card gate and with review.
The check assumes each tier key is its tier's build tag, as `PATTERN-test-speed` requires and nothing enforces: on a hub whose tier commands pass other `-tags` values, D7 demands the key as the tag.

### D8 — `doc.go` check in the plan

A card whose Create, Edit, Extends, Delete, Rename or Move targets touch non-test Go code of a package directory that holds a `doc.go` covers that `doc.go` when one of these holds:

- any of its own groups, Uses and Delete, Rename and Move included, names the `doc.go`;
- it targets the package self glyph;
- a later card names the `doc.go` under Edit or Extends.

Otherwise a `doc-go-unlisted` finding names the card and package, with the way forward: list it under Edit when the card changes what it says, under Extends when it only adds to it, under Uses when it stays as is and no later card targets it, else leave it to the later card that edits it.
A `Uses` listing records that the writer read the doc and found nothing the card falsifies.
A target is test code when it is a file target ending `_test.go`, a `_test` pseudo-unit, a member glyph that D4's resolving method reports declared only in `_test.go` files, or a member glyph that does not resolve yet and is a Rename new side whose old side is test code or names a member whose name begins `Test`, `Benchmark`, `Fuzz` or `Example`;
any other target, an ambiguous answer not split by build constraint included, is non-test code.

Bound: a `doc.go` listed under Extends whose text the card falsifies passes, and so does one a later card edits for some other reason; both stay with review.
A `doc.go` listed under Uses passes even when the card falsifies its text; review catches that.
A package the plan creates, a test-only card, a `**Prosa:**` or `**Custom:**` group and `docs/overview.md` are not checked.
A card creating a test helper by member glyph whose name begins with none of those prefixes is not test-only: the helper counts as non-test code, since a Create member glyph names no file, so the card lists the `doc.go`.

Rationale: the entry's "Intent changes a package's behaviour" is not machine-readable; a card changing a package's code is the proxy, and each way to cover the doc fits the plan's existing `uses-later-target`, `redundant-package-target` and `card-field-overlap` checks.

### D9 — Where the plan checks run

D6, D7 and D8 are blocking findings among the plan gate's context checks in `internal/loomshed`, beside `rename-tool-missing`, so `PlanContext` carries to them the tier tags as a value read from `gate.yaml`, as `RefuseRaw` is, the seam markers as a reader over the worktree root called at check time through the `impactset` marker parser.
Both `PlanContext` builders, `validate-plan`'s and the gate's in `internal/shedrecipe`, fill these fields the same way.
`planContextFindings` gains the index its callers already hold, for D6's declaring-file match and D8's classification.
They therefore run at Plan-Write's gate, a twill Plan-Write's gate and Plan-Burler-Fix's gate, and in `validate-plan`'s no-flag, `--rework`, `--lens-review` and `--overlay` modes; `--require-approved`, which loads no `PlanContext`, and webster's dispatch do not run them.
They skip the cards webster's run record holds done, as the other tree-dependent checks do, and they run only on a plan whose `language:` is `go`.

### D10 — Rules live in the specs; stencils point at them

`loom-discussion-spec.md` states D1 and D2 for loom and darn records and D3 and D4 for loom records: the heading form, the acceptance rule, which tokens are tested, the ` (new)` and ` (removed)` marks, the `<placeholder>` rule, and that a Go symbol is cited qualified or as a glyph.
`loom-plan-spec.md` states D6 to D8 in its validation checks and card rules.
The Discussion writer stencil and the Plan writer stencil each gain one sentence naming the checks the gate runs and pointing at the spec section, and restate no rule;
the chair stencil gets the Discussion sentence through its include of the writer stencil.
The darn writer stencil already points at the discussion spec and gains nothing.

### D11 — Help and PATTERN text

The `validate-discussion` and `validate-plan` help texts name the new checks.
`pattern/PATTERN-test-speed.md` gains a `//lym:seam` section beside its `//lym:guard` section, with the stacking rule of D5.

## Constraints

- `PATTERN-stencil-ownership` wins over the entry's "the writer stencils state the new rules": a stencil points at the format contract and never restates it, so the rules go in the specs and the stencils carry pointer sentences (D10).
- `PATTERN-sole-parsers`: the decision record is parsed only by `internal/discussionparser`, the plan only by `internal/planparser`; D3 to D8 consume their parsed output.
- `PATTERN-gate-slot` and the Gate Self-Check Parity Invariant in `internal/loomshed`: each gate and its verb call one function, so the new checks enter through `ValidateLoomDiscussion`, `ValidateDarnDiscussion` and the plan gate's context checks, never through the verb alone.
- `PATTERN-refusal-way-forward`: every new finding's text names its way forward, and a test reaches each one.
- `internal/discussionparser` stays stdlib-only, per its package doc; symbol resolution stays out of it (D4).
- `internal/planglyph` is the sole owner of quarry calls; D4 resolves through the `planindex.Index` seam the caller hands in.
- `PATTERN-durable-vs-ephemeral-state`: D3 names the `_lym` and `.lym` roots through `internal/lymdirs`, never as literals.

## Auto-mode assumptions

- The citation check is loom-only, a narrowing of the entry's "the gates of the rows that run it" (D4).
- A `pkg.Member.Sub` citation is tested by its `pkg.Member` prefix only, a further narrowing of the entry's "every Go symbol", since quarry gives a struct field no glyph (D4).
- Seam tests come from a marker over every reflected struct, wider than the entry's `Deps` and `Env` (D5).
- "Intent changes a package's behaviour" is read as "card touches the package's non-test Go code" (D8).
- A card is held to a marked struct when any target names it, not only when it adds a field, and a later card that edits the test covers it (D6).
- A new test file may state a `||` disjunction for a shared helper or `untagged` for a support-only file, wider than the entry's exactly one tier tag (D7).

## Open risks

- A loom run in Discussion or Plan review, or a darn run before its handoff, when this lands meets D1 to D8 at its next gate and takes one re-prompt to add ids, acceptance mentions, marks or listings.
- Every test reaching `envkit.NilSeams` marks the whole struct set `Env` nests, so a card touching any of those structs, their declaring files or their packages lists each such test under D6; a plan changing the wiring's dependency structs takes more seam-test listings than one changing `Env` alone.
- `landing-settles-in-place` changes `internal/loomcli/landingdeps_test.go`, a reflective test D5 marks; D5's marking runs against the parent at Plan time, and a reflective test a peer lands before this task needs the marker too.

<!-- lym:carry-over Discussion-Review -->
- Discussion-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/discussion/round-2-review.md; fixer report: _lym/reviews/discussion/round-2-fixer-report.md.
  - d4-struct-field-citation-has-no-glyph (class design, severity MEDIUM, open in rounds 2)
  - d5-marker-set-misses-nested-nilseams-structs (class design, severity MEDIUM, open in rounds 2)
  - d6-later-card-cover-unbounded (class design, severity MEDIUM, open in rounds 2)
<!-- /lym:carry-over Discussion-Review -->

## Acceptance criteria

- A record with an H3 under `## Decisions` that is neither `### D<n> — …` nor an `Added after Discussion` entry, or with a repeated id, fails both recipes' discussion checks with the finding naming the heading (D1).
- A record whose D3 appears in no acceptance bullet fails with `decision-not-in-acceptance` naming D3 and the way forward; D1 is not matched by D10 alone; `lym loom decision add` on a record with no D ids still succeeds (D2).
- A loom record citing a missing path under `internal/` fails with `citation-path-missing` naming the token and line; the same token marked ` (new)` passes; an import path, a `_lym/…` path, a token with a `<placeholder>` and a path inside a fenced block pass untested (D3).
- A loom record citing a missing qualified symbol or glyph fails with `citation-symbol-missing`; an existing qualified symbol and glyph pass, as does an existing qualified struct field such as `shedrecipe.Env.PlanIndex`; a fixture with two package directories sharing a base name yields `citation-symbol-ambiguous`, and a `testdata` directory does not; backticked headings, `C#`, a markdown anchor, `state.json` and `os.ReadFile` are untested; a quarry failure returns an error, not a pass; a darn record's citations are not tested (D4).
- Citations inside an `Added after Discussion` entry, a suffixed `## PATTERN changes` bullet or a carry-over block are not tested (D3, D4).
- A test reflecting over a struct with `NumField`, directly or through a same-package `_test.go` helper, without `//lym:seam` fails the scan; today's tests the scan sees carry markers; a test carrying `//lym:guard` and `//lym:seam` in either order passes both the guard scan and the seam parser; a `//lym:seam` marker whose glyph names a non-struct fails the scan; a test calling `envkit.NilSeams` without a marker, or with markers missing a struct `Env` nests, fails the scan (D5).
- A card adding a field to a marked struct, by member, handle, declaring file or package self glyph, without covering its seam test fails `validate-plan` with `seam-test-unlisted`, as does a card adding a field to a nested `Deps` struct such as `landingshed.Deps` without its seam test; listing it under Uses with no later card targeting it passes; a later card editing it passes; a `_test` pseudo-unit spelling matches (D6).
- A card creating a `_test.go` file with no Intent line stating its tier fails with `test-tier-unstated`; one tier tag passes; two tags without `||` fail; `untagged` passes; without `gate.yaml` the check is skipped (D7).
- A card editing production code in a package with a `doc.go` and not covering it fails with `doc-go-unlisted`; a Uses listing with no later editor, a later card's Edit of the doc, a package self glyph target and a Delete of the doc each pass; a test-only card and a new package pass, as does a card editing only a test function by member glyph in a package that also holds production code, and a card renaming a test function (D8).
- D6 to D8 fail the Plan-Write gate and each of `validate-plan`'s no-flag, `--rework`, `--lens-review` and `--overlay` modes, skip a done card and a `language: csharp` plan, and neither `--require-approved` nor webster's dispatch reports them (D9).
- The discussion and plan specs carry the rules; the Discussion writer and Plan writer stencils point at them without restating them, and the chair stencil carries the pointer once, through its include (D10).
- `lym loom validate-discussion --help` and `lym loom validate-plan --help` name the new checks, and `pattern/PATTERN-test-speed.md` describes the marker (D11).

## PATTERN changes

- `PATTERN-test-speed` — changes: Writing or running a test: via `lym gate test` by gate key; one `tierN` tag, the lowest that fits; it pins behavior no test covers; injected clocks, `t.Parallel` by default; a test reflecting over a struct's fields carries `//lym:seam`. (test)

## Notes for the plan writer

- `impactset`'s `//lym:guard` parser is the model for D5's marker parser.
