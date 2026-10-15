# The reed viewer follows the run

One batch, `01-follow-state-focus`, covered all six cards and recorded `done` at head `86339b93281a6c41ecf9c72536676292842aa535`.

The fork reports one commit per card:

- Follow state: the `@lyx_follow` session option decides whether a new agent window takes the viewer.
- Attach: selects the relevant window beside another client while follow is on.
- `Engine.FollowRun`: turns follow on and selects the relevant window.
- `lyx loom start`: every entry calls `FollowRun`; `lyx loom resume` does not.
- `lyx reed follow`: the new verb, told its geometry, turns follow on and selects the relevant window from tmux facts.
- Bindings and button: manual window-change bindings write follow off, and the follow button toggles it.

## Deviations

The scope guard listed four files changed outside the batch's own targets, informational only:

- `internal/loomcli/smoke_drivingsurface_test.go`
- `internal/reedengine/bindings.go`
- `internal/reedengine/bindings_integration_test.go`
- `internal/reedengine/bindings_test.go`

The fork also added `lyx reed follow` to the skip list in the reed command's root `PersistentPreRunE`, so the verb does not resolve the working directory outside a repository.

## Not run

The fork ran the batch gate (`lyx gate test --tags integration` over the four packages) green.
It did not run the real-tmux and smoke tests it changed or added, which are only compiled under `go vet -tags tmux` and `-tags llm`.
