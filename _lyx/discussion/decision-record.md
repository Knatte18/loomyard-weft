# Decision record — lens fan and twill fixer for Webster, Merge and Darn review

## Goal

Let Webster-Review, Merge-Review and Darn-Review run the Burler lens fan and the twill fixer per round, as Discussion-Review and Plan-Review already can, with no run changing until a hub sets a key.

## Scope

- New per-round fan keys `webster_fan`, `webster_fix_fan`, `merge_fan` and `merge_fix_fan` in `loom.yaml`, and `review_fan` and `review_fix_fan` in `darn.yaml`, each with the existing fan grammar and an empty template default.
- The wiring threads them into `RowClusterFans`, `RowFixFans` and `RowModelKeys` for the Webster, Merge and Darn rows.
- Fork reviewers and fix-check forks may run read-only git; the stencils say so.
- Two new fans in the embedded `burler.yaml`: `review-code` and `fix-code`, with code lenses.
- Recipe comments, `doc.go` files, config templates and `docs/overview.md` stop saying those three perches always run solo.

## Decisions

### D1: The fan grammar and its validation move into `burlerengine`

`FanList`, its shape error and the per-key fan validation move from `loomengine` into `internal/burlerengine`, and `loomengine` and `darnengine` both call them.
Rationale: darn needs the same grammar and refusals, `darnengine` cannot import `loomengine`, and both already import `burlerengine` or may (`burlerengine` imports neither engine).
The validation is told its module name, so each refusal names `loom config key` or `darn config key`.
The top-level-key-at-line lookup that names a malformed value's key moves to `configengine`, since both loaders need it.

### D2: Key names and placement

`loom.yaml` gets `webster_fan`, `webster_fix_fan`, `merge_fan` and `merge_fix_fan`; `darn.yaml` gets `review_fan` and `review_fix_fan`, matching darn's existing `review_*` budget keys.
Every new key is `CategoryGuard`, closed to the local layer as the four existing fan keys are, and an open map so reconcile and `--set` carry a list whole.
Each template default is `""`, which runs every round solo with the single fixer.

### D3: Wiring

`wire` adds the Webster and Merge Bouncer and review rows to `RowClusterFans`, and their review rows to `RowFixFans`, from the loom keys.
`wireDarn` adds the Darn rows from the darn keys.
`RowModelKeys` gains entries for the Webster, Merge and Darn review and fix rows: the review and fix model keys are Webster's segment keys (Merge and Darn already run on Webster's models), and the fix fan key is the segment's fix fan key.
A darn fix fan key names its file (`review_fix_fan in darn.yaml`), because its model keys are in `loom.yaml`.
Bound: with an entry present, the review half records each round's fan names before spawning, and the fix half refuses a round whose usage record lacks them, the same refusal Discussion and Plan rounds have; its way forward is a `goto` to the segment's review row and resume.
This newly reaches a Webster, Merge or Darn round reviewed by an older binary and fixed by this one.

### D4: Forks may run read-only git

The fork boilerplate in `burler-cluster-fan.md` and `burler-fix-self-check.md` changes from "never run any git command" to: read-only git only, such as `git diff`, `git log` or `git show`, and never one that changes the repository, its index, refs or working tree.
The engine's fork audit is unchanged and keeps failing a fork that runs a state-mutating git subcommand.
Bound: this lets a fork run git that reads; the audit's mutating-subcommand pattern still bounds what the stencil permits.

### D5: Code fans in `burler.yaml`

`review-code` holds the procedure lenses `code-correctness`, `code-affected-set`, `code-tests` and `code-pattern-fit`.
`fix-code` holds `fix-code-correctness`, `fix-consistency`, `fix-proposal` and `fix-affected-set`.
Each new review lens names the rubric item it emphasizes by its heading, without paraphrasing it, with "where this round's rubric carries it" when the three code rubrics differ.
Each review lens ends with "Your list:", as the other `review-*` lenses do.

### D6: The fan-conflict refusal stops naming `loom.yaml`

`burlerReviewEntry`'s refusal of a wiring fan beside a recipe-profile fan says the list came from the hub config through the wiring, since a darn row's list comes from `darn.yaml`.

## Constraints

- `PATTERN-config-layers`: only `configengine` reads layer files; the new keys load through each module's existing `Load`.
- `PATTERN-refusal-way-forward`: every new or moved refusal names its way forward and a test reaches it.
- `PATTERN-leaf-packages`: `internal/reviewbudget` stays untouched; no leaf package gains an import.
- `PATTERN-test-economy`: new cases are rows in the existing config, wiring and template tests.
- `PATTERN-documentation-lifecycle`: the built design lands in `doc.go` files.
- Defaults leave every hub's runs unchanged.

## Auto-mode assumptions

- The darn keys are named `review_fan` and `review_fix_fan`, with no `darn_` prefix, because the file already scopes them.
- Merge-Review and Darn-Review get their own fan keys rather than reading Webster's, because the board entry asks for the same keys per perch.
- The review and fix model keys a Merge or Darn row's engine refusal names are Webster's, the keys its models come from today.
- The `review-code` and `fix-code` lens texts are a first draft; later audits tune them.

## Open risks

- The read-only git permission rests on the stencil and the audit's subcommand pattern; a read-only-looking command outside that pattern that mutates state would pass the audit.
- Forks that run `git diff` over a large range spend more of their tool-call cap than the Discussion and Plan forks do; the 6-call cap is unmeasured for code review.
- A hub `burler.yaml` that defines a lens named like a new one shadows the new text through the per-name fallback.

## Acceptance criteria

- Each of the six new keys loads as a scalar or a per-round list, refuses an empty list, a non-scalar entry and an undefined fan naming its module, key and entry, and defaults to `""`.
- The six keys are closed to the local layer and carried whole as open maps.
- With the loom keys set, the wiring's `RowClusterFans`, `RowFixFans` and `RowModelKeys` hold the Webster and Merge rows' lists and keys; with the darn keys set, a darn wiring holds the Darn rows'.
- With every key at its default, the Webster, Merge and Darn rows run solo with the single fixer, as before.
- The shipped loom and darn recipes pass the gate-quiescence rule with fans set on every review perch.
- The embedded `burler.yaml` resolves `review-code` and `fix-code` to their lenses in order, and every fan names defined lenses.
- The fork stencils permit read-only git and still forbid mutating git; the audit still refuses a fork that runs a mutating git command.
- No recipe comment, doc or template line still says Webster-Review, Merge-Review or Darn-Review always runs solo or that forks may run no git.

## Notes for the plan writer

- `internal/loomengine/config.go` `validateFanKeys`, `FanList` and `keyAtLine` are the code D1 moves.
- `internal/loomcli/wiring.go` `reviewRowModelKeys` and `internal/loomcli/darnwiring.go` `wireDarn` are D3's sites.
