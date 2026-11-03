Task: check the correctness of the fix's changed code (the `removeFixTask` type assertion and `stepRemoveCommandParses`) from the review and the files in context.

no findings

- `removeFixTask` in `internal/adoptcli/green.go` now does `removal := err.(*fronttask.RemoveError)`.
  - `fronttask.Remove` (`fronttask.go:222-247`) returns only `nil` or a `*RemoveError`, at each of its four error returns (cancel ×2, record, entry).
  - `removeFixTask` reaches the assertion only when `err != nil`, so the assertion cannot panic.
  - A concrete failure, `HasRun` returning an error, gives `Step: "cancel"`, and the line is built from `removal.Step`, `removal.Err` and `HandRemoval()`.
  - `errors` is still used at `green.go:257`, `boardengine` by `entryDone`, and `fronttask` by the assertion, so no import is orphaned.
  - The gate's `./internal/adoptcli` build and tests pass.
- `stepRemoveCommandParses` (`internal/boardcli/verbs_test.go`, registered in `cli_test.go` after "FrontTask") seeds three entries and, per slug, strips `lym board remove '` and the trailing `'` from `RemoveCommand`. It then runs the real `remove` verb on the payload and asserts that `get` returns no task.
  - It fails with the defect in place. A bare-slug rendering such as `lym board remove 'fix'` leaves `fix` as the payload, so `resolveLookup` rejects it and `runJSON(t, 0, ...)` fails on the non-zero exit.
  - A payload with an unquoted or wrong key would likewise be rejected by `lookupPayload`.
  - A `RemoveCommand` that dropped the quotes or prefix fails the `HasPrefix`/`HasSuffix` guard.
  - The slugs `fix`, `adopt-fix-alpha-t1` and `a-b-c-d` are dashed names that pass the slug rule, and `%q` JSON-quoting of them matches `RemoveCommand`'s `json.Marshal` output.
  - Entry ids 0-2 are distinct, and each step runs after `fixture.reset`, so the seeded board does not leak into other steps.
- No caller of a changed signature exists, since no signature changed. `HandRemoval` and the `fronttask` refusal text still call `boardengine.RemoveCommand` unchanged.

Tool calls made: none. The context already held every changed file and the gate result.
