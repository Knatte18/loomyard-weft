# Decision record — twb-d0-ga: Plan and Webster checks out of step with the code a plan changes

## Goal

A plan whose later card moves a package that an earlier card of the same batch works in records done when the work is right, and its batch gate names the packages that exist after the batch.
A card that breaks a repo-wide scan test or a test that pins a symbol by name fails its own card gate, at the card that caused the break, instead of surfacing only at the plan-level verify or by hand.

## Scope

In:

1. Record-batch's and recover-batch's post-batch pass (the done-checks and handle binding) follow the directory moves that later cards of the same batch declare (D1–D3).
2. The card gates and the batch gate that begin-batch and recover-batch render name each directory as it stands after the moves of the batch (D4).
3. A webster config key names package arguments that every card gate runs beside the card's own packages (D5).
4. The docs listed under Constraints.

Out:

- Resolving each card's glyphs at that card's own commit.
- A plan-time check in `validate-plan` or Plan-Review that flags name-pinned tests or rename ordering.
- A verb that accepts a batch by hand.
- Drift detection and the glyph scope guard: both keep their present treatment of a moved package (a moved file is its Rename card's own target, so the guard reports no deviation for it).
- begin-batch's dispatch-time checks (`delete-before-reference`, the forthcoming exclusions) and the refusal routing that `plan-drift-refusal-route` covers.
- The plan-level `## verify:` command.

## Decisions

### D1. A directory move is declared by a Rename group's file pairs

A Rename group declares a directory move from `D` to `D'` when its file-rename pairs whose old side lies directly in `D` all have their new side directly in `D'`.
No move is declared when those pairs land in two or more directories, when `D` or `D'` is the repository root, or when one of `D` and `D'` lies inside the other.
Symbol rename pairs declare no move.
`internal/planparser` derives the moves of a card list from plan data alone, since it owns Rename pairs; consumers never re-derive them.

Rationale: the `tender` plan spelled its package rename exactly this way, one file pair per file, and the plan format has no other spelling for a package rename.
A split, root or nested move has no single unambiguous destination to follow.

### D2. Remapping a glyph through moves uses the sanctioned converters

A glyph or `plan:` handle body whose unit path is `D` is remapped to unit `D'` with its member unchanged: the unit path is read through `Glyph.UnitPath`, the new unit is built through `glyph.Self`, and the result is spelled through `Glyph.String`.
An external-test unit `D_test` is remapped to `D'_test`: `goPackageDir` maps it to `D`, and a new sanctioned inverse beside it in `internal/planparser` builds `D'_test`.
This changes `PATTERN-glyph-conversion-chokepoint`: the single sanctioned adjustment of a Go unit string becomes the external-test mapping in both directions, and the PATTERN line and its background change in the same commit.
A file self glyph is remapped by its own file pair, not by the directory move.
A glyph whose unit is not a moved directory is unchanged.

Rationale: without the `_test` inverse, a Create in an external test package of a moved directory fails the done-check exactly as the `tender` cards did.

### D3. The post-batch pass checks each card at its final location within the batch

For card `c` of a batch, the done-checks and handle binding remap every Create, Delete and Rename target through the directory moves of the cards after `c` in the same batch, applied in card order so chained moves compose, and check the remapped key only.
Handle binding matches the remapped expected glyph against the batch delta's created and renamed-to sides, as it already matches the spelled one, and binds the handle to the glyph it matched, which names the destination directory.
A finding on a remapped key names the spelled ref, the remapped key and the card whose move it followed.
Moves declared by cards in other batches never apply.
recover-batch gets this through the post-batch pass it already shares with record-batch.

Bound: a target is followed only through a move a later card of the same batch declares, and only to that move's destination.
A Create whose symbol the move dropped still fails `create-not-done`, and a Delete whose symbol survives at the destination still fails `delete-not-done`, so following never passes work missing from the batch's final tree.
The moving card's own `rename-not-done` check still proves the files moved.

Rationale: the batch's final tree is what lands, so checking there is stronger than checking each card's own commit, and the declared move tells the check where the work went.

### D4. Gate scopes replace a moved-out directory by its destination

A Rename group's own pairs already reach a gate only through their new side, so the change concerns the other targets cards place in a moved directory `D`.
In the batch gate, each card's directory `D` that a move of that card or a later card in the batch takes out is replaced by `D'`, applied in card order.
`D` stays beside `D'` when it holds a `.go` file in the worktree at render time that no pair of the move takes out, or when a card of the batch Creates a file target in `D` that no pair moves.
In a card gate, a directory `D` that a move of the batch takes out is replaced by `D'` only when `D` holds no `.go` file at render time, which is the recovery case of a move that already landed; otherwise the card gate keeps `D`, since it runs before any later card's move.
The disk reads use the worktree root the render is handed (`PATTERN-told-geometry`).
Moves apply to gate scopes under every plan language, `none` included, since they are path-based.

Bound: a directory leaves a gate only through a declared move, and only when no Go file of it stays behind, so a gate never drops a package that still exists.

Rationale: `lyx gate test` refuses a package directory that no longer exists, so the `tender` batch gate failed on `./internal/pausecli` and `./internal/pauseengine`, which cards 8–10's targets placed in its scope.
A recovery render after the move would otherwise drop `D` from an earlier card's gate silently through the existing no-`.go`-file rule.

### D5. A config key adds fixed packages to every card gate

webster's config gains `card_gate_packages`, a string of whitespace-separated root-module `go test` package arguments, empty in the template, classified `CategoryGuard`, read afresh at every begin-batch and recover-batch as `kickstart_pack` is.
Every card gate renders one more untagged `lyx gate test` step over those entries, after the card's own package steps and before the two lints, for every card, a docs-only card included, since `cmd/lyx` holds the docs link and PATTERN scans a docs card can break.
An entry the card's own root-module scope already names is left out of that step, and a step with no entry left is not rendered.
Config load refuses an entry that is not `./` followed by slash-separated segments of letters, digits, `_`, `-` and `.`, or that has an empty, `.`, `..` or `...` segment, with a message naming the key, the entry, and `lyx config webster --set card_gate_packages=…` as the way forward, as the role model-spec check already fails the load.
An entry whose directory holds no `.go` file in the worktree at render time is left out, and begin-batch or recover-batch returns a warning naming it and the same way forward.
The key is Go-scoped; a hub that leaves it empty, every non-Go hub included, renders card gates unchanged.
The batch gate is unchanged by the key.
This repo's value, `./cmd/lyx`, is pinned on the hub's board-layer webster config by `lyx config webster --set card_gate_packages=./cmd/lyx` after landing; the hub reconcile gives the board layer the template's empty value until then.

Bound: the step runs only the listed packages, untagged, and an entry outside the package-argument shape fails the load, so the key cannot turn a card gate into a whole-module run or carry anything but a package path into the rendered command.
A listed package that no longer exists stops being run per card, with a warning on every render, and the plan-level verify still runs it.

Rationale: the name-pinned test (`cmd/lyx/gitrepoboundary_test.go`) broke on a method the card itself added, and `TestTierPurity_UntaggedTestsSpawnNothing` on a new `time.Sleep`; no plan-time check can see either, but running the package that holds them catches both.
The card gate, not the batch gate, catches the break at the card that causes it, where the fork holds that card's context.
The list is config, not code, because which packages hold repo-wide scans is a property of the repo; a string, not a list, because `--set` and every webster key are scalar.
`CategoryGuard` keeps a local layer from blanking the step.

## Constraints

- `PATTERN-glyph-conversion-chokepoint` is changed as D2 states; otherwise D2's remap uses only `glyph.Self`, `Glyph.UnitPath` and `Glyph.String`.
- `internal/planparser` stays quarry-free: D1 and D2 use only quarry's `glyph` package there, never a resolve.
- `PATTERN-ref-shape-registry` and `PATTERN-sole-parsers`: classifying a pair as a file pair and reading Rename groups stay in `internal/planparser`.
- `internal/websterengine` imports neither `internal/planglyph` nor quarry and reaches resolve-backed work through `planindex`.
- `PATTERN-config-layers` and `PATTERN-config-strictness`: the key is declared and classified in `ConfigSpec` and loaded through the existing `LoadOrTemplate` path.
- `PATTERN-refusal-way-forward`: D5's load refusal names the way forward, and gets a refusal-spec row and a test if the role model-spec load error it mirrors has one.
- `PATTERN-test-speed` and `PATTERN-test-economy`: untagged unit tests over hand-built plans and fake deltas, extending the existing done-check, bind, gate-scope and config tables; only behavior that needs a real resolve uses the existing `planglyph` resolve fixtures.
- Docs land in the same commit as the code: `internal/websterengine/doc.go` (done-check and gate sections), `contracts/specs/loom-plan-spec.md` (Tier 1 names the configured packages and drops "the card gate does not build or test the rest of the module" for them; Tier 2 names the post-move directories), `internal/websterengine/template.yaml`, and the PATTERN line and background D2 changes.

## Auto-mode assumptions

- A1. Not asked: following declared moves over resolving per card commit (D3).
- A2. Not asked: moves come from file-rename pairs only (D1).
- A3. Not asked: scan packages run per card and come from config (D5).
- A4. No plan-time check is added for name-pinned tests, since the cited breaks named nothing that existed at plan time.
- A5. Not asked: the operator pins this repo's value on the board layer after landing (D5).

## Open risks

- `lyx gate test ./cmd/lyx` took 8 s wall time in one warm-cache run; a change to a package `cmd/lyx` imports relinks its test binary, so the per-card cost on a cold build is unmeasured, and the extra step adds load on the hub's capped gate slots shared by concurrent pairs.
- Quarry's delta may report a moved file's symbols as created at `D'`, as renamed to `D'`, or as neither; binding (D3) covers the first two, and a fixture over a real delta settles which.

## Acceptance criteria

- The `tender` shape: earlier cards Create and Edit members in `D`, the batch's last card moves every file of `D` to `D'`; record-batch records the batch done, binds each handle to its `D'` glyph, and the rendered batch gate names `D'` and not `D`.
- The earlier cards' card gates name `D` at begin-batch; at a recovery render after the move landed, they name `D'`.
- A Create in the external test package of `D` is checked at `D'_test`.
- Chained moves `D`→`D'`→`D''` in two later cards of the batch check an earlier card's target at `D''`; a move into a `D'` that already holds Go source checks the target at `D'`.
- No move is declared for pairs out of `D` that go to two destinations, for the repository root, or for nested `D` and `D'`; the done-checks look up the spelled key.
- The batch gate keeps `D` beside `D'` when `D` keeps a `.go` file no pair moves.
- A Create whose symbol is absent from `D'` after the move fails `create-not-done`, naming the spelled key, the remapped key and the moving card.
- A Delete whose symbol is present in `D'` after the move fails `delete-not-done`.
- A move declared by a card in a later batch does not apply to the current batch's checks or gates.
- recover-batch records a recovered batch of the `tender` shape done.
- A `language: none` plan applies moves to its gate scopes.
- `card_gate_packages` empty: every rendered card gate is unchanged.
- `card_gate_packages` set: every card gate, a docs-only card's included, renders one extra untagged `lyx gate test` step over the entries before the lints, without an entry the card's own scope names.
- An entry outside the package-argument shape (absolute, `..`, `...`, whitespace or shell metacharacters, a leading `-`) fails config load naming the key and entry; an entry whose directory holds no `.go` file is left out and named in a warning.
- A local layer cannot set `card_gate_packages`.

## Notes for the plan writer

- The `tender` plan's card 12 (`12-tender-rename.md` in the `loomyard-weft` history, commit `0fa48607`) is the shape to reproduce in fixtures.
