# lym board set-status releases a project with its draft member tasks

`lym board set-status '{"slug":"<project>","status":null}'` on a project whose own status is empty now also releases every direct member task in `draft`, in the same board write.
The parent's dobby then starts those members in their `depends_on` order, as it starts any released task.
After an arras-style filing, the operator releases the whole family in one step instead of releasing each member by hand.

## What changed

- `internal/boardengine/projectrelease.go` adds `Board.SetStatusReleasingProject`, which only the `set-status` verb calls.
  - Any non-null status, and any entry that is not a project, are handled exactly as before.
  - A null status on a project with an empty status (nil or `""`) releases the project and every member task whose `project` names it and whose status is `draft`.
    It returns the released slugs in store order.
  - Each member is released through `Store.SetStatus`, so `validateWrite` judges it as its own release would.
    The critical section's run-lock, project and slug checks then judge the whole write.
    When any member fails, the error names the member and the in-memory mutation is dropped unsaved, so nothing is released.
  - A null status on a project whose status is a run status or `done` is refused.
    The refusal names a `lym board set-status '{"slug":"<member>","status":null}'` line for each draft member, or says that none exists.
- `internal/boardcli`'s `set-status` verb calls the new method.
  When members were released, the success envelope carries a `notice` naming them.
  The verb's help text and the package doc describe the project release.
- `Board.SetStatus`, `Store.SetStatus`, upsert, batch and merge are unchanged and release no member.
  So batten cancel's claim clear, and every other null-status writer, still clears only the entry it names.
  The `# Draft` section of boardengine's package doc now says so.

## Design decisions

- **Hand-set project status.**
  The record names two cases: an empty project status, which releases the family, and a run status or `done`, which is refused.
  A project carrying any other status, such as a hand-set word, takes the null as a plain clear and releases no member.
  This keeps the trigger to the empty-status case the record names.
- **Members untouched by the release.**
  A member that is already released, run-held or done stays as it is.
  So do a nested project and its members, and every entry outside the project.
  Only direct members of kind task in `draft` count.
- **Sequential validation is safe.**
  Members are released one after another against the same store.
  A release changes only a status, which none of `validateWrite`'s checks read from other entries.
  So each member's result is the one its own release would get.

## Tests

- `internal/boardengine/projectrelease_test.go` (tier 0):
  - a project release on a mixed family releases only the direct draft members;
  - a member failing validation, a run-status project and a done project are each refused, with nothing written;
  - `SetStatus`, an upsert and a project with a hand-set status release no member.
- `internal/boardcli`'s `TestCLI` gains the `ProjectRelease` step, which pins the verb's wiring and its notice.
