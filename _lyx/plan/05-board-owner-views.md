# Card 5 — board-owner-views

**Edit:**
- `internal/boardengine#BriefTask`
- `internal/boardengine#Store.ListTasksBrief`
- `internal/boardengine#renderReadme`
- `internal/boardcli#outputGetTask`
- `internal/boardcli#storeVerbs`
- `internal/boardengine_test#TestListTasksBriefLayerAndProposal`
- `internal/boardengine_test#TestRenderReadmeGolden`
- `internal/boardcli_test#TestCLI`

**Create:**
- `internal/boardengine#Board.EffectiveOwner`

**Uses:**
- `internal/boardengine#Owners.Effective`
- `internal/boardengine#ParseOwners`

**Intent:**
D9's board views.
`BriefTask` gains `OwnerMachine string` (`owner_machine`), which `Store.ListTasksBrief` fills on each task with its effective owner (the recorded one, else `Owners.Effective` over the store's own entries and `parentOf`); a note carries none, and a resolver error leaves the cell empty rather than failing the list.
`Board.EffectiveOwner` loads the store and returns the slug's effective owner.
`lyx board get` (the get verb in `storeVerbs`) sets the returned task's `OwnerMachine` to `Board.EffectiveOwner` when none is recorded, before `outputGetTask` writes it; `outputGetTask` keeps its envelope shape.
`renderReadme`'s Running table gains an `Owner` column after `At`, holding the entry's recorded `owner_machine`, empty when none is recorded, so the README's Running rows name the owner.
`TestListTasksBriefLayerAndProposal` gains assertions: a task with an owner lists it, an unowned task lists the prime owner, a note lists none, and a store whose `parentOf` fails lists an empty owner and still succeeds.
`TestRenderReadmeGolden`'s running-task row gains the Owner column with a recorded owner.
`TestCLI` gains steps over a board whose `board.json` is seeded with a top-level `prime_owner` written straight into the file, since `SetOwners` is not the subject here: `lyx board get` of an unowned task prints the prime owner as `owner_machine`, and `lyx board list` carries `owner_machine`.

**ImpactSummary:** read-only views of boardengine and boardcli; the README's Running table gains one column, pinned by its golden.

**Commit:** 5: board: show each task's owner in list, get and the README
