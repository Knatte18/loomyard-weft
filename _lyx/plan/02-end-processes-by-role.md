# Card 2 — end-processes-by-role

**Create:**
- `plan:internal/runproc#EndRole` -> `func EndRole(dir, owner, role string, callerPID int) ([]Ended, error)`

**Edit:**
- `internal/runproc#ender.end` -> `func (e ender) end(dir, owner string, callerPID int, ends func(role string) bool) ([]Ended, error)`
- `internal/runproc#End`
- `internal/runproc#EndSparing`
- `internal/runproc#TestEnd`
- `pattern/PATTERN-run-process-owner.md`
- `PATTERN.md`

**Extends:**
- `internal/runproc/doc.go`

**Intent:**
D4 ends a beam's dobby through `runproc` by its `RoleDobby` record alone, never by ending every process of the beam's slug, which would end the pawl run's driver and agents too; `runproc` has only `End`, `EndSparing` and `EndProcess`, so it gains `EndRole`.

`EndRole` ends the live records of `role` under `owner` in `dir`, and the processes below them, with `End`'s signal sequence, grace, kill bound, re-listing passes and `SurvivorsError`, skipping `callerPID` and its ancestors; every record of another role, and every process below such a record, is left alone exactly as `EndSparing` leaves a spared role.
`ender.end` takes the role selection as a predicate instead of a spare list: `End` passes one that ends every role, `EndSparing` one that ends every role not in its spare list, and `EndRole` one that ends only `role`; a record the predicate does not select is treated as `EndSparing` treats a spared one today, so `End` and `EndSparing` behave as before.
On Windows `EndRole` signals nothing, as `End` does.

`TestEnd` gains a `role` field driving `EndRole`: a row whose records hold a `dobby` record and a `driver` record under one owner ends only the dobby and its descendants and leaves the driver's process and its group unsignalled; the existing spare row keeps passing through `EndSparing`.
`runproc/doc.go` names `EndRole` beside `End` and `EndSparing`.
`pattern/PATTERN-run-process-owner.md`'s background rewrites its `End`-only lines, the opening sentence's "a run's processes are ended only through `runproc.End`", "`End` alone ends the processes an owner holds" and the review line "each end goes through `End`": they name `End`, `EndSparing` and `EndRole` as `runproc`'s ends of an owner's records, `EndRole` ending one role's records under an owner with `End`'s sequence, as the decision record's constraint on this background permits (D4).
`PATTERN.md`'s entry becomes the declared line:

```markdown
- `PATTERN-run-process-owner` — A run's processes: registered in `internal/runproc`, ended only via `runproc`'s `End`, `EndSparing` or `EndRole`. — [background](pattern/PATTERN-run-process-owner.md)
```

The new verb skips no guard: it ends a subset of what `End` ends for the same owner, bounded by the role name.

**ImpactSummary:** one new export and an internal re-sign whose only callers are `End` and `EndSparing` in the same file; `TestEnd`'s existing rows keep their expectations; `PATTERN.md`'s `PATTERN-run-process-owner` line and its background name the new end beside `End` and `EndSparing`.

**Commit:** 2: runproc: end the records of one role under an owner
