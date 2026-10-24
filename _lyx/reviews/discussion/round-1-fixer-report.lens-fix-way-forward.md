```yaml
lens: fix-way-forward
outcome: reported
```

Lens: fix-way-forward. I checked that each way forward the fix added can be carried out from the state it names and makes progress.

1. **MEDIUM**, D10, changed text: "a task with no pair on this machine, and a task another machine owns, naming `lyx adopt <slug>`".
   For a task that was never started, this way forward goes nowhere.
   Evidence: `internal/adoptcli/doc.go` (Step 2) says adopt adds a pair only for "each named task that holds an owner or a run status".
   A task never spawned holds neither, so adopt adds no pair, and a re-run of promote is refused again for having no pair.
   The refusal needs a separate way forward for the unspawned case, for example `lyx board upsert` of the entry as a project and then `lyx beam create`.
   Adopt fits only when the task's branches are on origin.

2. **MEDIUM**, D7, changed text: "`lyx adopt <beam>` when it does, whose pair then carries the origin record, beam mark included, from the beam's weft branch".
   No Decision says adopt accepts a project entry, so this way forward may not be reachable.
   Evidence: `internal/adoptcli/doc.go` admits a "named task", refuses a note, and its Step 2 adds pairs for tasks only.
   D1 extends only "the owner setter, `Owners.Effective` and the list view's owner" to projects, not adopt.
   The second half of the claim holds: `internal/fabricengine/doc.go:596` keeps an adopted weft's origin record.
   The way forward still needs adopt to admit a project entry, and only D8 or D1 can state that.

3. **MEDIUM**, D8, changed text: "The guard refuses every other worktree … naming the pair's parent worktree from its origin record, or the prime when it records none, as where to run the verb."
   The named parent may be a place where the same guard refuses again.
   D5 exposes `lyx fabric add <slug> --parent <slug>` for any pair of the hub, so a pair's recorded parent can be a task pair that is not a beam, and the guard refuses there too.
   The recorded parent worktree can also be gone; D12 itself handles "a pair whose parent worktree is gone".
   The way forward should name the nearest beam or the prime up the origin chain, or the prime outright.

4. **LOW**, D9, changed text: "a beam another machine owns names the owner and `lyx beam create <slug>` run there".
   If the owner machine is gone or will never run the create, nothing on this machine finishes the stopped create.
   D3 gives `lyx beam create` as the only way to set the flag, and Finding 2 leaves adopt of a project unstated.
   Name the takeover route as a second way forward.

5. **LOW**, D9, changed text: "an absent entry or a note names `lyx board upsert` of a project entry".
   For a note, it is unclear whether that upsert can reuse the note's slug.
   That would be an upsert changing a note's kind to project, and D14's list no longer includes kind changes.
   No Decision says an upsert may change a note's kind.
   Upserting under a new slug does make progress, so this is only ambiguous.

Tool calls:
- Read `internal/adoptcli/doc.go` lines 1–80, to check what adopt admits and adds; my context held only its line 25.
- Grepped `internal/fabricengine/doc.go` and `add.go` for origin-record handling, to verify D7's claim that an adopted pair keeps the beam's origin record; my context lacked Add's handling of a branch already on origin.
