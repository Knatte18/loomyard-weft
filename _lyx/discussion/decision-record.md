# Decision record — plan-glyph-precision

## Goal

Plan cards name the members they change as glyphs instead of file paths, so webster's guards, plan validation and every downstream agent work at member granularity.
Before the stencil pushes every plan onto member glyphs, loomyard's own use of quarry glyphs (the plan stencil, the handle rules, the containment checks) is tested systematically and every defect found in that use is fixed in this task.
No check this task adds or changes gives a different verdict at dispatch than it gave at the plan gate because of state the run itself changes; a check whose verdict depends on such state runs at the plan gates only.

## Scope

In:

- A systematic, committed test of loomyard's glyph chain over the member shapes real plan cards name: parse, resolve, Create inversion, handle canonicalization, handle binding through the delta, done-checks, ref rewrite, and the target checks this task adds or changes.
  Defects it exposes in loomyard code are fixed in this task.
- Retiring the cross-card containment refusals (`containment-unit-overlap`, `containment-file-overlap`).
- A same-card redundancy check: a card listing a file or package and a member of that same file or package.
- Retiring `handle-unreferenced`.
- A re-sign marker on `**Edit:**` member bullets, and a caller-coverage check for deleted and re-signed members.
- The plan stencil (`contracts/stencils/loom/loom-template-plan.md`): skeleton and rules for member-glyph `Edit:` targets, handles for every new member, the re-sign marker, caller coverage.
- The Plan-Review rubric (`contracts/stencils/loom/loom-rubric-plan-review.md`): a finding for an `Edit` card holding only Go file paths of non-test files while its Intent names specific symbols, and one for an `Edit` member whose Intent changes its signature without the re-sign arrow.
- `contracts/specs/loom-plan-spec.md`, `internal/planglyph/doc.go`, `internal/planparser` doc comments and any other spec row or doc naming a retired or added check ID.

Out:

- Kick-start packs (`kick-start-pack`, `enclose-kickstart`) and any use of quarry `Enclose`.
- Changes to quarry itself; a defect rooted in quarry is recorded, not fixed (see Decisions).
- Parallel card execution (`webster-parallel-execution`).
- Webster's changed-files guard and `ScopeGuard`: they already map member targets to files and symbols.

## Decisions

### D1 — The glyph-use test comes first and stays committed

Decision: The plan's first cards build a table-driven test over a committed fixture Go module in testdata, one row per member shape class, driving each row through loomyard's own glyph chain end to end: `planparser` parse, `planglyph` validation (resolve, Create inversion, the D3 redundancy check, the D5 caller coverage), `CanonicalizeHandles`, `BindHandles` over a real commit's delta, `DoneChecks`, and `RewriteRefs`.
The shape classes are harvested by the Plan producer from a sample of real plans in the weft history (at least the three the board cites: `gate-list`, `rework-generations`, `test-suite-change-amplification`), and at minimum cover: package-level func, type, var, const; a method on a value and on a pointer receiver; a struct field; an interface method; a generic type and its method; a member declared in a `_test.go` file; a member in a build-tagged file; a member in a package whose directory name differs from its package name; a member of a `main` package under `cmd/`; a member of a var or const block; a `plan:` handle created in one card and listed by later cards beside its file (the `rework-generations` shape).
Every card leaves the suite green: a row lands in the same card as the code that makes it pass, and no card commits a failing row.
The test card's rows assert the chain as it stands; rows for D2's retired containment, D3 and D5 are added or flipped by those decisions' own cards.
The Plan producer finds loomyard defects before writing the plan, by driving every shape class through the chain during exploration in a probe it does not commit.
Each defect found gets its own fix card, ordered after the test card and before the stencil card, which adds the row the defect breaks.
A row that fails because of quarry itself asserts quarry's current answer, carries a comment naming the gap, and is named in the task's summary for the operator.
Rationale: The board orders the test before the stencil change, and a committed fixture keeps the coverage after the task; real plans' live trees drift and cannot be a test input (`card-fabric-reference`).

### D2 — Cross-card containment is retired

Decision: `containment-unit-overlap` (`planparser`'s syntactic tier) and `containment-file-overlap` (`planglyph`'s resolve tier) are removed as cross-card checks, with their spec rows, doc entries and tests.
Bound: two cards whose targets overlap physically are no longer refused at any gate.
What bounds it: webster runs batches one at a time in one worktree, each batch to one fork that works its cards in plan order, batches come back in plan card order, `CheckBatchOrder` asserts that order, and `recover-batch` re-runs the same batch's cards rather than pairing different cards; an overlap between two sequential cards is harmless, which is what `rework-generations` batch 5 showed.
Rationale: The checks guarded against blind parallel dispatch, which webster no longer does, and the resolve tier changed verdict mid-run because a `plan:` handle has no file until it is built.

### D3 — Same-card redundancy is a plan-gate finding

Decision: A new blocking check flags a card whose own targets list both a container and a member inside it: a package self glyph and a member of that package, a package self glyph and a file of that package, or a file and a member that resolves into that file.
The package-level pairings are pure and live in `planparser`; a `Create` handle counts as a member of the unit it declares, and a `Rename` to-side handle as a member of its pair's Old glyph's unit, the unit `CanonicalizeHandles` writes into it.
The file-versus-member pairing needs the resolve answer and lives in `planglyph`, reading the member's file from the resolved `Symbol.File`.
`CanonicalizeHandles` runs inside `resolvePass` at every gate and leaves a handle a handle; it becomes a plain member glyph only when record-batch's `BindHandles` binds it, into the same unit.
The pure pairings run wherever `planparser.ValidateFormat` runs, `ValidateDispatch` included: their verdict is the same before and after binding, so they need no gate split.
The file-versus-member pairing runs in a resolve-backed pass that `ValidateFormat` and `Validate` call (and through them `ValidateRework` and the verbs that call them) and `ValidateDispatch` does not; it is not added inside `resolvePass`, which `ValidateDispatch` shares.
Bound: dispatch never refuses on the file-versus-member pairing, so one that only appears after record-batch binds a handle into a member glyph is let through; it is harmless, since a redundant listing is plan noise that changes no card's work.
The finding's message names the way forward: keep the member glyphs and drop the container, or keep the container when the card changes the whole file.
Rationale: The board wants this flagged at plan time; running the file-versus-member pairing at dispatch would reintroduce the verdict change D2 removes.

### D4 — `handle-unreferenced` is retired

Decision: The check is removed, with its spec row; the stencil sentence requiring every handle to be used by a later card is D6's.
Every new member a card creates is declared as a `plan:` handle, whether or not a later card references it.
`handle-dangling`, `handle-collision` and `handle-malformed` stay.
Bound: a declared handle no other card uses is now legal; it is still bound at record-batch (`bind-count-mismatch`) and done-checked (`create-not-done`), so a handle the card fails to build is still caught.
Rationale: The check forced planners to invent a glyph for a new symbol only its own card uses, the spelling the handle rule exists to prevent.

### D5 — Re-sign marker and caller coverage

Decision: An `**Edit:**` member bullet may carry the arrow form `` `<glyph>` -> `<new declaration head>` ``, marking a signature change and stating the new head.
The head is parsed like a `Create` head (one Go declaration, one symbol) and must name the same member as the glyph; a mismatch or parse failure is a blocking finding.
The arrow therefore applies only to a member one Go declaration names: a package-level func, type, var or const, or a method.
An interface method or a struct field has no declaration of its own, so every head written for it fails to parse or names another member; such a re-sign carries no arrow and states the change in its card's Intent.
Bound: a re-signed interface method or struct field gets no caller coverage; for those the coverage would only be informational (see the over-reporting bound below), and webster's per-card build and test gate fails the card that breaks a caller.
`planparser` parses the arrow into its glyph and head; the head-to-member match runs inside `CanonicalizeHandles`, whose one batched `quarry.Name` call takes each head as a second declaration source, under the unit of its glyph.
Each head's answer is compared with its bullet's glyph and then discarded: it never enters handle canonicalization, the canonical-collision check or the `RewriteRefs` rewrite, since the head names an existing member, not a handle.
`CanonicalizeHandles` runs inside `resolvePass`, so the match runs at every gate, `ValidateDispatch` included, and reads only plan text, so its verdict cannot change mid-run.
A receiver change that keeps the member path (value to pointer) names the same member; one that changes the receiver type names another member and is a mismatch.
The arrow on a target that is not a member glyph (a file path, a file or package self glyph, or a `plan:` handle, whose member has no prior signature) is a blocking finding.
The arrow's only mechanical effect is marking the member for caller coverage; the head is parse-checked and member-matched, and nothing downstream compares it with the built declaration.
Bound: a head that names the right member but misstates the signature passes; it misleads only the implementer reading the card, and the card's Intent and webster's per-card build and test gate bound it.
A new blocking check, caller coverage, runs at the plan gates: for every `Delete` member glyph and re-signed `Edit` member, each reference to it in the tree outside its own declaration's resolved span, the declaring file included, must be covered by a target (the file holding it, that file's package self glyph, or a member glyph whose resolved span holds the reference).
A `Delete` member's caller may be covered by the same card or an earlier card, which can remove the reference first.
A re-signed member's caller must be covered by the same card: no earlier card can call a signature that does not exist yet, and a card that changes the declaration without its callers breaks the build D1 requires every card to keep green.
On the card that deletes or re-signs the member, D3 rules out the declaring file and the package self glyph beside the member, so there a caller is covered only by a member glyph whose span holds the reference or by another file of the package or of another package; a package self glyph covers a `Delete` member's callers only from an earlier card.
A re-sign held by two cards is checked per card against the plan-time tree, so each card covers the callers.
A `Rename` Old side is not checked: the spec's Rename mechanic is an AST-aware rewrite that updates every reference itself, followed by a grep pass for leftovers, so its callers are the card's own work without being listed.
Bound: a `Rename` card's callers are bounded by that mechanic, its grep verify and webster's per-card build and test gate; the spec's existing caveat for string- and reflection-based references stands.
A `Delete` target that is a file path, a file self glyph or a package self glyph is not checked either: its members' callers are let through, bounded by webster's per-card build and test gate, and for a package by `delete-before-reference`'s import match against later cards' Edit regions.
For a `Delete` member, a reference inside a later card's Edit region keeps the existing `delete-before-reference` finding and is not reported twice; for a re-signed member, which `delete-before-reference` does not cover, such a reference is a caller-coverage finding.
Reference detection is a new walk over every `.go` file of the worktree's tree, `_test.go` files and files under any build tag included, `testdata` and `vendor` directories excluded; non-Go files are not searched, and a file an earlier card creates is not in the tree at plan time and is not searched.
The walk tokenizes each file with `go/scanner`, so comments and string literals never match: a package-level member is referenced by an identifier token of its name not preceded by `.` in its own package, and by the tokens `<package clause name>` `.` `<Name>` elsewhere; a method or field (a member path with a `.`) by the tokens `.` `<Name>` anywhere.
A member's own package is the files of its declaring directory whose package clause equals the declaring file's; an external test package in that directory (`package <name>_test`) is elsewhere and is matched by the qualified form.
`deleteorder.go`'s line-textual patterns, which also match inside comments, strings and selectors, are not reused for this check.
Bound, over-reporting: the scan is name-based, so for a method or field it cannot tell receivers apart; for those the finding is informational.
For a package-level member it is blocking, and in its own package it still matches a local identifier, a struct-literal key or a method declared under the same name; such a finding is cleared by a cover the card can carry under the rules above, which on the deleting or re-signing card itself means the member glyph whose span holds the occurrence, even when that member is not otherwise changed.
Bound, under-reporting: a cross-package reference is matched only as `<package clause name>.<Name>`, so a caller importing the package under an alias or with a dot import, and a caller in a file an earlier card creates, are let through; webster's per-card build and test gate fails the card that breaks such a caller, and a planner who knows of one covers it like any other caller.
The check runs in the plan-gate-only pass D3 names, never in `ValidateDispatch`, so a reference set that shrinks as earlier cards land cannot change a verdict mid-run.
Rationale: The board wants deletion and re-sign caller coverage mechanical once cards name members; the format had no way to say which `Edit` member changes its signature, and the arrow reuses the grammar `Create` and `Rename` already carry.

### D6 — Stencil and rubric

Decision: The plan stencil's skeleton shows an `**Edit:**` card listing member glyphs, plus a test file path, and its text says: `Edit:` lists the member glyphs the card changes; a file path only for a file with no symbol to name (a test file the card adds tests to, a non-Go file, or a whole file the card creates or deletes).
The stencil drops the "used by at least one later card" rule (D4), adds the re-sign arrow and the caller-coverage obligation (D5), and points at the spec for the redundancy check (D3) rather than restating it (`PATTERN-producer-pointer-rule`).
The Plan-Review rubric gains a finding: an `Edit` card whose targets are only Go file paths of non-test files while its Intent names specific symbols; the finding names the member glyphs to list instead.
It gains a second: an `Edit` member glyph without the re-sign arrow while the card's Intent changes that member's parameters, results, type parameters or receiver; the finding names the arrow to add, and never fires for an interface method or a struct field, which take no arrow (D5).
Rationale: Agents copy the shape they are shown, and the mechanical gates cannot tell from a file path which members the Intent means.

### D7 — Card order inside the plan

Decision: The plan orders its cards: D1's fixture and test, fixes D1 exposes, D2, D4, D3, D5, then D6's stencil and rubric last.
Per D1, the `rework-generations` row and the containment rows flip in D2's card, the `handle-unreferenced` rows in D4's, the redundancy rows land in D3's and the arrow and caller-coverage rows in D5's.
Rationale: The board requires the glyph-use test before the stencil change, and the stencil must describe checks that already exist.

### Added after Discussion (parent, 2026-10-08): Rename Old side outside caller coverage

Decision: Caller coverage does not apply to a Rename Old side: the Rename mechanic rewrites every reference and the build gate confirms it. Caller coverage applies to Delete members and re-signed members only; the spec's Rename text stands unchanged.

Rationale: The spec's Rename rewrites callers itself, so requiring caller files as same-card targets is a burden with no failure it prevents.

### Added after Discussion (parent, 2026-10-08): File and package Delete outside caller coverage

Decision: Caller coverage checks Delete member glyphs only. A Delete by file path or package self glyph, and a file Rename Old side, are out of scope; D5's under-reporting Bound names this bypass, bounded by the existing delete-before-reference check for packages and by verify's build.

Rationale: Keeps the new check to member targets, the shape this task introduces; whole-file and package deletes stay with the existing checks and the build.

### Added after Discussion (parent, 2026-10-08): No re-sign arrow for interface methods and fields

Decision: The re-sign arrow is written only for members quarry.Name accepts as one Go declaration (funcs, methods, types, vars, consts). Interface methods and struct fields take no arrow, and D6's missing-arrow rubric finding exempts them.

Rationale: quarry.Name cannot parse a bare field or interface method, so requiring the arrow there is unsatisfiable.

### Added after Discussion (parent, 2026-10-08): Own package means same package clause

Decision: A member's own package is the files in its declaring directory whose package clause equals the declaring file's. An external _test package in that directory falls under the qualified pkg.Name rule.

Rationale: External test packages reach the member qualified; the bare-identifier rule would miss them.

## Constraints

- `PATTERN-glyph-conversion-chokepoint`: every glyph-to-path step uses `Glyph.UnitPath` or the resolved `Symbol.File`; no trimming or regex over a glyph string.
- `PATTERN-ref-shape-registry`: the new checks classify refs through `planparser`'s kind-policy ledger, each with its own gate constant and ledger entry; retired gates leave the ledger.
- `PATTERN-sole-parsers`: the re-sign arrow is parsed by `planparser` only.
- `planglyph`'s chokepoint pin (`TestChokepointCallSites_PinnedToTheirGuardedFunctions`): `quarry.Name` is called only from `CanonicalizeHandles` and `Resolve` only from `resolveTargets`; no new check adds a call site elsewhere.
- `PATTERN-refusal-way-forward`: each new blocking finding names its way forward, gets its spec row in the same commit, and a test reaches it.
- `PATTERN-gate-self-check-parity`: the plan gate and `lyx loom validate-plan` keep calling the same function, so the new checks reach both.
- `PATTERN-producer-pointer-rule`: the stencil points at `loom-plan-spec.md` for check definitions.
- `PATTERN-test-economy`, `PATTERN-test-speed`, `PATTERN-test-isolation`: the D1 test spawns git, so it sits in the `integration` tier under `gitkit.HermeticGitEnv()`; existing tests that cover a row are extended rather than duplicated.
- `PATTERN-documentation-lifecycle`: check lists live in `internal/planglyph/doc.go`, `internal/planparser` doc comments and `loom-plan-spec.md`; no design doc is added.
- `PATTERN-comment-line-breaks` and semantic line breaks in every `.md` file.
- A test fixture comes from committed testdata, never from the weft repo or another worktree.

## Auto-mode assumptions

- `bugfix-webster` has landed: it is off the board, and the card-order checks it added (`CheckBatchOrder`, `delete-before-reference`) are on this branch.
- The board's "a file path only for a file with no symbol to name (tests, non-Go files)" is read as: a test file the card adds tests to, not a test function the card changes in place, which is a member like any other.

## Open risks

- D5's same-package over-report on a package-level name can force a planner to list a file or member it does not change, and on the deleting or re-signing card itself D3 leaves only member glyphs and other files as covers.
- D1 may expose more loomyard defects than this task can carry, which can make the plan large.
- Planners may skip the re-sign arrow; D6's missing-arrow rubric finding is the only backstop, and it catches only a signature change the Intent states.

## Acceptance criteria

- The D1 test runs every listed shape class through parse, validation, canonicalization, binding, done-checks and rewrite, and passes; any quarry-rooted row asserts the current answer with a comment naming the gap.
- The `rework-generations` shape (a card creates a handle in a package; a later card lists a file of that package and the handle; a third card lists the same file) gets no blocking finding at the plan gate and none at a later `begin-batch` after record-batch has bound the handle.
- Two cards targeting the same file, or a member and its file, on different cards, validate clean.
- A card listing a file and a member resolving into that file, or a package self glyph and a member or file of that package, gets the redundancy finding at the plan gate, with its way forward.
- A card listing a file and a `plan:` handle built in that file gets no redundancy finding at the plan gate, and none from `ValidateDispatch` once record-batch has bound the handle to its glyph; a card listing a package self glyph and a `plan:` handle of that package gets the redundancy finding at the plan gate, and from `ValidateDispatch` once the handle is bound.
- A card listing a package self glyph and a `Rename` whose to-side `plan:` handle spells a different unit than its Old glyph's gets the redundancy finding exactly when the Old glyph's unit is that package, at the plan gate and from `ValidateDispatch`; in particular, a self glyph of the package the draft handle spells, with the Old glyph in another unit, gets no redundancy finding at either.
- A `plan:` handle declared on a `Create` card and referenced by no other card validates clean, binds at record-batch and passes its done-check.
- An `Edit` arrow bullet whose head parses to the glyph's member validates; a head naming another member, one that fails to parse, or an arrow on a file path, self glyph, `plan:` handle, interface method or struct field target is a blocking finding.
- The same member re-signed by arrow bullets on two cards, each card covering the callers, validates clean, with no `handle-canonical-collision`, and no arrow causes a rewrite of the plan on disk.
- Deleting or re-signing a package-level member whose caller no card D5 admits covers is a blocking caller-coverage finding naming the file; a caller in the declaring file outside the member's span is included; covering it by another file, or by a member whose span holds the reference, on the same card, or by file or package self glyph on an earlier card for a `Delete`, clears it; a method or field case is informational.
- A caller covered only by an earlier card clears a deleted member's caller-coverage finding, but not a re-signed member's.
- Renaming a member, and deleting a file or package, gets no caller-coverage finding.
- A same-package occurrence of a package-level member's name only inside a comment, a string literal or after a `.` gets no caller-coverage finding; a `pkg.Name` reference in an external test package of the declaring directory is a caller.
- A reference to a deleted member held by a later card's Edit region still yields `delete-before-reference` alone, not a second caller-coverage finding; the same reference to a re-signed member is a caller-coverage finding.
- `ValidateDispatch` reports no caller-coverage finding, including after an earlier card lands and the reference set shrinks.
- No spec, doc, stencil or test still names `containment-unit-overlap`, `containment-file-overlap` or `handle-unreferenced` as a live check.
- The plan stencil's skeleton shows member-glyph `Edit:` targets, and the Plan-Review rubric carries the file-paths-only and missing-arrow findings.
- `go build ./...`, `go vet` under the `integration`, `tmux` and `llm` tags, `go test ./...` and `go test -tags integration ./...` pass.

## Notes for the plan writer

- Cross-card containment lives in `internal/planparser/containment.go` and `internal/planglyph/containment.go`; their pairing machinery can be reused for D3 by comparing within one card instead of across cards.
- `handle-unreferenced` is in `checkHandleConsistency` (`internal/planparser/validate.go`).
- `delete-before-reference` (`internal/planglyph/deleteorder.go`) is called from `resolvePass` in `internal/planglyph/planglyph.go`, which `ValidateDispatch` shares; D5's caller coverage goes in the separate plan-gate-only pass, not beside that call.
- Real plans for the D1 sample are in the weft repo's history, under `_lyx/plan/`; read them during exploration only.
