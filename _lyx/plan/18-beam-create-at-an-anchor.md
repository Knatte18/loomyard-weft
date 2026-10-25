# Card 18 — `lyx beam create --anchor <dir>`

**Edit:**
- `internal/fabricengine#AddOptions`
- `internal/fabricengine#Topology.add`
- `internal/beamcli#create` -> `func create(deps createDeps, slug, anchor string) (createResult, error)`
- `internal/beamcli#createSteps` -> `func createSteps(deps createDeps, rec *fabricengine.Mutations, entry boardengine.OwnerEntry, here, slug, anchor string, result *createResult) error`
- `internal/beamcli#createDeps`
- `internal/beamcli#createCmd`
- `internal/beamcli#promoteProject`
- `internal/beamcli#hubCreateDeps`
- `internal/beamcli/doc.go`
- `internal/beamcli#TestCreate`
- `internal/beamcli#TestCreate_StepFailureNamesTheRerun`
- `internal/beamcli#createWorld.deps`
- `internal/lyxcwd#ResolveWithAnchor`
- `docs/shared-libs/lyxcwd.md`

**Create:**
- `internal/fabricengine#Topology.EnsureAnchor`
- `internal/fabricengine_test#TestAdd_AtAnchor`

**Uses:**
- `internal/fabricengine#validateAnchorTarget`
- `internal/fabricengine#Topology.BeginAnchorChange`
- `internal/fabricengine#AnchorChange.Apply`
- `internal/fabricengine#AnchorChange.Finish`
- `internal/fabricengine#AnchorChange.Close`
- `internal/fabricengine#moveWeftContent`
- `internal/fabricengine#MoveRecord`
- `internal/fabricengine#moveLockPath`
- `internal/fabricengine#MoveRecordPath`
- `internal/fabricengine#RemoveMoveRecord`
- `internal/lyxcwd#NamedWorktree`

**Intent:**
D6. The flag reaches only the pair being created: `AddOptions` gains `Anchor string`, set only by `lyx beam create --anchor`; `lyx fabric add` and `lyx beam promote` take no such flag.
`Topology.add` with `Anchor` validates it with `validateAnchorTarget` against the fork point's warp commit, before any mutation, refusing an invalid, untracked or link-crossing value with nothing created; each refusal carries `validateAnchorTarget`'s text, the next step card 15 names for it.
After the fork, and before any wiring or other write at the new anchor, it takes the move lock and writes the move record of the new pair (`A` the fork point's anchor, `B` the flag's value, kind move, its pid and start time), then runs `moveWeftContent(rec, weft, A, B, names)` on the new pair's weft, the one step-4 move `AnchorChange.Apply` also runs; `BeginAnchorChange` and `Apply` are not called, so their committed-marker and dirty-weft refusals do not apply to the freshly forked weft, which is clean, and a fork point with no committed marker, as on a hub not yet migrated, is moved from its resolved anchor and gets the marker `B` committed.
Then it builds the new pair's Location with `lyxcwd.ResolveWithAnchor(target, B)` and uses it for the wiring, scratch link, launchers and origin record, and removes the move record through `RemoveMoveRecord` before it returns, after those steps, so ahead of `createSteps`' push, workspace and dobby steps, which card 16's gate would refuse while the record stands.
This reads D6's "once the remaining create steps are done" as Add's own remaining steps, since `createSteps`' later steps cannot run while the record stands.
An error between the record's write and its removal leaves the record standing and is returned naming the re-run `lyx beam create <slug>`, which finds the pair present and recovers the record through `EnsureAnchor`.
A re-run after a kill past the record's removal finds the pair present and the pair's anchor `B`, and passes `EnsureAnchor`'s equal-anchor branch.
An `Anchor` equal to the fork point's anchor is no move.

`Topology.EnsureAnchor(rec, pair, anchor)` serves a re-run whose pair is present: with a standing move record it first recovers through `BeginAnchorChange` with the record's `B`, `Apply` and `Finish`, the re-run path `lyx fabric anchor` takes (undoes an uncommitted move and moves again, or, after the commit, goes on); an `Apply` error, `landed` or not, is returned naming `lyx beam create <slug>` again as the re-run, which recovers the record the same way, and `Close` releases the move lock; then an `anchor` equal to the pair's anchor, or an empty one meaning the pair's own, rewires the junctions and scratch link and rewrites the launchers idempotently, and a different one is refused naming `lyx fabric anchor <slug> --set <anchor>` and then `lyx beam create <slug> --anchor <anchor>` again, which passes as the equal case and finishes the mark, push, workspace and dobby steps.
A held move lock is refused with `MoveInProgress`'s text.

`beamcli`: `createCmd` gains `--anchor <dir>` (help: a path relative to the warp repo root, validated in the commit the beam forks from; without it the beam inherits its parent's anchor), `create` and `createSteps` carry it, `createDeps.AddPair` passes it in `AddOptions.Anchor`, and on every present pair, with or without an anchor, `createSteps` calls the new `createDeps.EnsureAnchor func(rec *fabricengine.Mutations, slug, anchor string) error`, which `hubCreateDeps` fills over `NamedWorktree(prime.HubPath, slug)`, before the mark, so a re-run without `--anchor` over a pair with a standing record recovers it before `MarkBeam` commits into the weft; the workspace regeneration and dobby start follow as today.
A nil `EnsureAnchor` is a wiring error in the hub filler only; the test fake fills it.
`beamcli/doc.go` documents the flag.

Tests:
`TestCreate` gains rows, with `createWorld.deps` recording `AddOptions.Anchor` and every `EnsureAnchor` call: `--anchor` reaches `AddOptions.Anchor`; on a present pair it calls `EnsureAnchor` with the anchor, and its refusal surfaces with no later step run; on a present pair with no anchor `EnsureAnchor` is called with `""`. `TestCreate_StepFailureNamesTheRerun` passes the new argument.
`TestAdd_AtAnchor` (tier1, hubforge) first commits `y/.keep` in the prime's warp, so `y` is a directory git tracks, and covers: a beam added with `Anchor: "y"` has marker, weft content, junctions, scratch link and launcher at `y` and no record left; a beam added with `Anchor: "y"` from a fork point whose weft has no committed marker (removed with `git rm` and a commit in the prime's weft) has the marker `y` committed and its content at `y`; an invalid, an untracked (`absent`) and a link-crossing anchor each refuse with nothing created, naming the next step `validateAnchorTarget` gives; `EnsureAnchor` on the present pair with `y` skips the move and restores a removed junction, with `""` does the same, and with `z` refuses naming `lyx fabric anchor` and `lyx beam create`; a kill before the move commit, built by hand on a pair added with no anchor (the record written with `A` `.` and `B` `y` and the content half moved with `git mv`, uncommitted), and a kill after it, built on a pair added at `y` with the record written back, are each finished at `y` by `EnsureAnchor` with no record left.

`promoteProject`, which runs `create` for a project entry, passes an empty anchor.
`ResolveWithAnchor`'s doc comment and `docs/shared-libs/lyxcwd.md`'s section on it name `Add`'s new pair at a given anchor as its third caller, standing at a worktree root above the anchor as clone does.

**ImpactSummary:** `create` and `createSteps` gain an argument with their four callers on this card (the command, `promoteProject` and two tests); `AddOptions` and `createDeps` gain one field each; a zero `Anchor` keeps today's add, and a re-run over a present pair now passes through `EnsureAnchor`, which with no record and an empty anchor only rewires idempotently; `ResolveWithAnchor`'s doc and its shared-libs section gain the new caller.
