```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens. It traces each change in the diff to the board entry's goal points and flags anything unrequested or anything left unbuilt.

**Decisions (the record has only the board entry; there are no `Added after Discussion` entries):**
- **G1 (Idea):** when the build changes and uncommitted paths block a hand-over, the old binary must not render the new board copies. The entry offers three routes:
  1. The old driver renders from its own embedded stencils until the hand-over.
  2. The old driver waits for the producer to commit, then hands over before the next render.
  3. A render that names an include its binary lacks hands over to a fresh driver instead of failing the run.
- The `## Seen` section describes the incident and asks for nothing.

**Changes and what each traces to:**
- **C1** (`internal/sheddriver/drive.go`): `handOver` loses its `DirtyPaths` refusal, `HandOverSeam.DirtyPaths` is removed, and so is the `strings` import. This builds G1 by route 3. The old step's failed render now reaches the existing hand-over row (`driverules.go:264`) and spawns a successor instead of parking. That row is reached for any binary-eligible stop (`driverules.go:203-215`), and the Producer kind is one of them.
- **C2** (`internal/loomcli/drivewiring.go`): the `DirtyPaths` wiring and `dirtyPathsOutsideRecords` are removed. C1 forces this, since the field it sets is gone. `loomengine.PathsOutsideRunRecords` keeps its caller at `internal/battencli/wire.go:1000`.
- **C3** (`internal/sheddriver/drive_test.go`): the dirty-tree row and the `dirty` field are removed, and "on a clean tree" is dropped from the hand-over row's name. C1 forces this, because the seam field no longer exists and the row's premise is stale.
- **C4** (`internal/loomcli/doc.go:646-649` and the `handOver` doc comment at `drive.go:345-353`): the clean-tree wording is rewritten. C1 forces this, since the old text stated the precondition C1 removes.
- **C5** (commits `995ed298` and `5c462bd1`): line breaks in the comments C3 and C4 changed. These touch only lines the earlier changes edited, so they are forced too.

**Each decision and the changes that build it:**
- G1: C1, with C2–C5 forced by it.

**Findings:** none.
- No change goes beyond what G1 asks or what a traced change forces.
- G1 is built by route 3, one of the routes the entry offers.
- The entry's route 3 is keyed on a render that names a missing include. C1 hands over at any binary-eligible stop after a build change, whatever the tree holds. That wider trigger is the hand-over row the code already had, so the diff only removes the block the entry names. It adds no new trigger, so this is not unrequested scope.
- No `Pattern-Departure:` trailer is needed.

Tool calls: none. The inherited context already held the diff, the record, the commits, `driverules.go` and the caller search for `PathsOutsideRunRecords`.
