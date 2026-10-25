# Card 3 — runproc-dobby-role

**Edit:**
- `internal/runproc#ender.end` -> `func (e ender) end(dir, owner string, callerPID int, spare ...string) ([]Ended, error)`
- `internal/runproc#End`
- `internal/runproc#TestEnd`
- `internal/runproc/doc.go`

**Create:**
- `plan:internal/runproc#RoleDobby` -> `const RoleDobby = "dobby"`
- `plan:internal/runproc#EndSparing` -> `func EndSparing(dir, owner string, callerPID int, spare ...string) ([]Ended, error)`

**Intent:**
Add the dobby role and an end that spares named roles, which `lyx reed down` of a beam's session needs so the beam's dobby, registered under the beam's slug, outlives its session (D1).
`RoleDobby` joins the role constants in `record.go`.
`ender.end` takes the roles to spare: a live record whose role is in `spare` is left out of the signalled set exactly as the caller and its ancestors are, on every re-list pass, and its descendants are not reached through it.
`End` calls `end` with no spare role, so every existing caller ends what it ended; `EndSparing` passes the roles through.
The package doc lists `RoleDobby` among the roles and states that `EndSparing` is `End` with named roles left alone, the one caller being the session end that is not a teardown.
`TestEnd` gains a row over its scripted process table: an owner with a batten record and a dobby record, ended with `RoleDobby` spared, signals the batten's process only and returns it alone; a second row ends the same owner with `End`, no role spared, and signals both, which is the teardown's end of a landed beam's slug reaching its dobby.

**ImpactSummary:** End keeps its signature and behaviour; the variadic spare reaches only the new EndSparing, so no existing caller changes.
