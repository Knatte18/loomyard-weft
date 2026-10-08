# Decision record — plan-glyph-precision

## Goal

Plan cards name the members they change as glyphs instead of file paths, so webster's guards, plan validation and every downstream agent work at member granularity.
Before the stencil pushes every plan onto member glyphs, loomyard's own use of quarry glyphs (the plan stencil, the handle rules, the containment checks) is tested systematically and every defect found in that use is fixed in this task.
Plan validation gives the same verdict at Plan-Write and at dispatch.

## Scope

In:

- A systematic, committed test of loomyard's glyph chain over the member shapes real plan cards name: parse, resolve, Create inversion, handle canonicalization, handle binding through the delta, done-checks, ref rewrite, and the target checks this task adds or changes.
  Defects it exposes in loomyard code are fixed in this task.
- Retiring the cross-card containment refusals (`containment-unit-overlap`, `containment-file-overlap`).
- A same-card redundancy check: a card listing a file or package and a member of that same file or package.
- Retiring `handle-unreferenced`.
- A re-sign marker on `**Edit:**` member bullets, and a caller-coverage check for deleted, renamed and re-signed members.
- The plan stencil (`contracts/stencils/loom/loom-template-plan.md`): skeleton and rules for member-glyph `Edit:` targets, handles for every new member, the re-sign marker, caller coverage.
- The Plan-Review rubric (`contracts/stencils/loom/loom-rubric-plan-review.md`): a finding for an `Edit` card holding only file paths while its Intent names specific symbols.
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
Every row that fails because of loomyard code is fixed in a card of this plan, ordered before the stencil card.
A row that fails because of quarry itself asserts quarry's current answer, carries a comment naming the gap, and is named in the task's summary for the operator.
Rationale: The board orders the test before the stencil change, and a committed fixture keeps the coverage after the task; real plans' live trees drift and cannot be a test input (`card-fabric-reference`).

### D2 — Cross-card containment is retired

Decision: `containment-unit-overlap` (`planparser`'s syntactic tier) and `containment-file-overlap` (`planglyph`'s resolve tier) are removed as cross-card checks, with their spec rows, doc entries and tests.
Bound: two cards whose targets overlap physically are no longer refused at any gate.
What bounds it: webster runs cards strictly one at a time in one worktree, batches come back in plan card order, and `CheckBatchOrder` asserts that order; an overlap between two sequential cards is harmless, which is what `rework-generations` batch 5 showed.
Rationale: The checks guarded against blind parallel dispatch, which webster no longer does, and the resolve tier changed verdict mid-run because a `plan:` handle has no file until it is built.
`webster-parallel-execution` would reintroduce parallelism in separate worktrees, with its own merge gate; it is a parked note and designs its own guard if built.

### D3 — Same-card redundancy is a plan-gate finding

Decision: A new blocking check flags a card whose own targets list both a container and a member inside it: a package self glyph and a member of that package, a package self glyph and a file of that package, or a file and a member that resolves into that file.
The package-level pairings are pure and live in `planparser`; a `plan:` handle counts as a member of its declared unit there.
The file-versus-member pairing needs the resolve answer and lives in `planglyph`, reading the member's file from the resolved `Symbol.File`.
Both run in the plan gates (`ValidateFormat`, `Validate`, `ValidateRework` and the verbs that call them) and never in `ValidateDispatch`.
Bound: dispatch never refuses on redundancy, so a redundancy that only appears after `begin-batch` canonicalizes a handle into a glyph is let through; it is harmless, since webster's changed-files guard covers both entries.
The finding's message names the way forward: keep the member glyphs and drop the container, or keep the container when the card changes the whole file.
Rationale: The board wants this flagged at plan time; running it at dispatch would reintroduce the verdict change D2 removes.

### D4 — `handle-unreferenced` is retired

Decision: The check is removed, with its spec row and the stencil sentence requiring every handle to be used by a later card.
Every new member a card creates is declared as a `plan:` handle, whether or not a later card references it.
`handle-dangling`, `handle-collision` and `handle-malformed` stay.
Bound: a declared handle no other card uses is now legal; it is still bound at record-batch (`bind-count-mismatch`) and done-checked (`create-not-done`), so a handle the card fails to build is still caught.
Rationale: The check forced planners to invent a glyph for a new symbol only its own card uses, the spelling the handle rule exists to prevent.

### D5 — Re-sign marker and caller coverage

Decision: An `**Edit:**` member bullet may carry the arrow form `` `<glyph>` -> `<new declaration head>` ``, marking a signature change and stating the new head.
The head is parsed like a `Create` head (one Go declaration, one symbol) and must name the same member as the glyph; a mismatch or parse failure is a blocking finding.
A new blocking check, caller coverage, runs at the plan gates beside `delete-before-reference`: for every `Delete` member, `Rename` Old side and re-signed `Edit` member, each file in the tree that references it, other than its own declaration, must be covered by a target of the same card or an earlier card (the file itself, its package self glyph, or a member glyph whose resolved span holds the reference).
A reference inside a later card's Edit region keeps the existing `delete-before-reference` finding and is not reported twice.
Reference detection reuses the existing `deleteorder.go` identifier scan.
Bound: the scan is name-based, so for a method or field (a member path with a `.`) it cannot tell receivers apart; for those the finding is informational, and blocking only for package-level members.
The check runs at plan gates only, never at dispatch, so a reference set that shrinks as earlier cards land cannot change a verdict mid-run.
Rationale: The board wants deletion and re-sign caller coverage mechanical once cards name members; the format had no way to say which `Edit` member changes its signature, and the arrow reuses the grammar `Create` and `Rename` already carry.

### D6 — Stencil and rubric

Decision: The plan stencil's skeleton shows an `**Edit:**` card listing member glyphs, plus a test file path, and its text says: `Edit:` lists the member glyphs the card changes; a file path only for a file with no symbol to name (a test file the card adds tests to, a non-Go file, or a whole file the card creates or deletes).
The stencil drops the "used by at least one later card" rule (D4), adds the re-sign arrow and the caller-coverage obligation (D5), and points at the spec for the redundancy check (D3) rather than restating it (`PATTERN-producer-pointer-rule`).
The Plan-Review rubric gains a finding: an `Edit` card whose targets are only Go file paths of non-test files while its Intent names specific symbols; the finding names the member glyphs to list instead.
Rationale: Agents copy the shape they are shown, and the mechanical gates cannot tell from a file path which members the Intent means.

### D7 — Card order inside the plan

Decision: The plan orders its cards: D1's fixture and test, fixes D1 exposes, D2, D4, D3, D5, then D6's stencil and rubric last.
Rationale: The board requires the glyph-use test before the stencil change, and the stencil must describe checks that already exist.

## Constraints

- `PATTERN-glyph-conversion-chokepoint`: every glyph-to-path step uses `Glyph.UnitPath` or the resolved `Symbol.File`; no trimming or regex over a glyph string.
- `PATTERN-ref-shape-registry`: the new checks classify refs through `planparser`'s kind-policy ledger, each with its own gate constant and ledger entry; retired gates leave the ledger.
- `PATTERN-sole-parsers`: the re-sign arrow is parsed by `planparser` only.
- `PATTERN-refusal-way-forward`: each new blocking finding names its way forward, gets its spec row in the same commit, and a test reaches it.
- `PATTERN-gate-self-check-parity`: the plan gate and `lyx loom validate-plan` keep calling the same function, so the new checks reach both.
- `PATTERN-producer-pointer-rule`: the stencil points at `loom-plan-spec.md` for check definitions.
- `PATTERN-test-economy`, `PATTERN-test-speed`, `PATTERN-test-isolation`: the D1 test spawns git, so it sits in the `integration` tier under `gitkit.HermeticGitEnv()`; existing tests that cover a row are extended rather than duplicated.
- `PATTERN-documentation-lifecycle`: check lists live in `internal/planglyph/doc.go`, `internal/planparser` doc comments and `loom-plan-spec.md`; no design doc is added.
- `PATTERN-comment-line-breaks` and semantic line breaks in every `.md` file.
- A test fixture comes from committed testdata, never from the weft repo or another worktree.

## Auto-mode assumptions

- No operator answered; every decision above is a self-pick recorded in the support log's question ledger.
- `bugfix-webster` has landed: it is off the board, and the card-order checks it added (`CheckBatchOrder`, `delete-before-reference`) are on this branch.
- Retiring cross-card containment is not a PATTERN change: no PATTERN entry names it.
- The board's "a file path only for a file with no symbol to name (tests, non-Go files)" is read as: a test file the card adds tests to, not a test function the card changes in place, which is a member like any other.

## Open risks

- The name-based reference scan behind D5 over-reports for common identifiers; blocking findings are limited to package-level members, but a package-level name shared with a local identifier in the same package can still force a planner to list a file it does not change.
- D1 may expose more loomyard defects than this task can carry; every one exposed is still fixed here (D1), which can make the plan large.
- A quarry-rooted failure found by D1 leaves a known gap the plan cannot close.
- Planners may skip the re-sign arrow; the Plan-Review rubric is the only backstop for an unmarked signature change.

## Acceptance criteria

- The D1 test runs every listed shape class through parse, validation, canonicalization, binding, done-checks and rewrite, and passes; any quarry-rooted row asserts the current answer with a comment naming the gap.
- The `rework-generations` shape (a card creates a handle in a package; a later card lists a file of that package and the handle; a third card lists the same file) gets no blocking finding at the plan gate and none at `begin-batch` after the handle is canonicalized.
- Two cards targeting the same file, or a member and its file, on different cards, validate clean.
- A card listing a file and a member resolving into that file, or a package self glyph and a member or file of that package, gets the redundancy finding at the plan gate, with its way forward; the same plan gets no redundancy finding from `ValidateDispatch`.
- A `plan:` handle declared on a `Create` card and referenced by no other card validates clean, binds at record-batch and passes its done-check.
- An `Edit` arrow bullet whose head parses to the glyph's member validates; a head naming another member, or one that fails to parse, is a blocking finding.
- Deleting, renaming or re-signing a package-level member whose caller file no same-or-earlier card covers is a blocking caller-coverage finding naming the file; covering it by file, package self glyph or a member whose span holds the reference clears it; a method or field case is informational.
- A reference held by a later card's Edit region still yields `delete-before-reference` alone, not a second caller-coverage finding.
- No spec, doc, stencil or test still names `containment-unit-overlap`, `containment-file-overlap` or `handle-unreferenced` as a live check.
- The plan stencil's skeleton shows member-glyph `Edit:` targets, and the Plan-Review rubric carries the file-paths-only finding.
- `go build ./...`, `go vet` under the `integration`, `tmux` and `llm` tags, `go test ./...` and `go test -tags integration ./...` pass.

## Notes for the plan writer

- Cross-card containment lives in `internal/planparser/containment.go` and `internal/planglyph/containment.go`; their pairing machinery can be reused for D3 by comparing within one card instead of across cards.
- `handle-unreferenced` is in `checkHandleConsistency` (`internal/planparser/validate.go`).
- The identifier scan to reuse for D5 is in `internal/planglyph/deleteorder.go`; its plan-gate call site is in `internal/planglyph/planglyph.go`.
- Webster runs cards one at a time: see `internal/websterengine/doc.go` ("Why cards run one at a time") and `internal/websterengine/sequence.go`.
- Real plans for the D1 sample are in the weft repo's history, under `_lyx/plan/`; read them during exploration only.
