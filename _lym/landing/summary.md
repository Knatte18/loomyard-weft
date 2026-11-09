# Release a project's draft member tasks with `lym board set-status`

`lym board set-status` with a null status on a project whose own status is empty now releases, in the same write, every direct member task of that project that is in `draft`.
Dobby then starts the members in their `depends_on` order, as it starts any released task.
This keeps the operator's step after an arras run to one command: the release of the project releases its family.

The success envelope's `notice` names the released members.
The verb's long help and the `boardcli` and `boardengine` package docs describe the behaviour.

## Design decisions

- The release lives in a new `Board.SetStatusReleasingProject` (`internal/boardengine/projectrelease.go`), which only the `set-status` verb calls.
  `Board.SetStatus`, `Store.SetStatus`, upsert, batch and merge release no member, so other null-status callers, such as batten cancel's claim clear, are unchanged.
- Only direct member tasks in `draft` are released.
  Members already released, run-held or done, a nested project and its members, and entries outside the project stay as they are.
- Each member is released through `Store.SetStatus`, so it is validated as its own release would be.
  One failing member refuses the whole write, and the refusal names the project and the member.
- A null status on a project with a run status or `done` is refused.
  The refusal names `lym board set-status` for each draft member as the way forward, or says there is no draft member to release.
- A project with any other hand-set status, a non-null status, and a null status on a task all take the plain `SetStatus` path and release no member.

## To check by hand

Nothing was run by hand.
The new tests cover the release, the refusals and the writes that release no member, in `internal/boardengine/projectrelease_test.go`, and the notice in `internal/boardcli`.
