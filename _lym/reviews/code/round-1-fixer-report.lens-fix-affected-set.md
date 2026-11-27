```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set check, rerunning the searches behind the set-shaped findings and checking the fix covers every member.

Unchecked findings: none of the set-shaped ones (F1, F2, F3/F10, F6, F8, F11). F4, F5, F7 and F9 have no set premise.

1. **LOW** — `internal/fabriccli/roleguard.go:5` (file header; F3/F10 push-admission set)
   - The header still says a detached push passes for "a darn writer's bench pair".
   - Push now also needs the docs darn run record and no batten run directory naming the slug.
   - The twin sentence on `injectedPathsAreOwn` (line 207) was updated to "A darn writer on a docs run"; this one was not.
   - The run-directory bar is stated in `fabriccli/doc.go`, but in neither header nor `injectedPathsAreOwn`.

2. **LOW** — `internal/pairteardown/leftover.go`, new `EndSubjectProcesses(prime, slug, false)` call (F1)
   - `EndSubjectProcesses` is documented as ending processes for "a subject another machine now owns" (`endsubject.go`, `pairteardown/doc.go:160`). It also logs `pairteardown: ended the processes of a subject another machine owns`. A bench-pair removal now writes that false log line, and neither doc covers this caller.
   - Its spare list (`runproc/end.go` `subjectPassSpared`: dobby, teardown, adopt, sync, push, slack relay) differs from the teardown's own `endOwnedProcesses`, which spares only teardown and orch. So an orch registered under a bench slug would now be ended, which the normal teardown never does.
   - The spared roles are still ended later by `td.Run`, so the new comments' "live processes are ended first" is not true for them.

3. **LOW** — `internal/battenshed/doc.go:108` and `internal/battencli/wire.go` Push closure (F2)
   - The new text puts the bench removal in "the push".
   - The same doc section says "With git or pushes off nothing is pushed" (`battenshed/doc.go` ~117), and `PATTERN-pair-remote.md` says "With git or pushes off it pushes nothing".
   - `RemoveBenchPairs` runs in that Push regardless of `SyncOptions`, and `archiveOwnerTip` (`fabricengine/ownerarchive.go`) pushes archive tags without reading them.
   - The tag push under pushes-off predates this fix, when it ran in Remove. The fix now places it inside the very step those sentences describe, with no exception stated.

The other set members hold:
- **F1:** the claims that a live process is ended, in `leftover.go`, `pairteardown/doc.go` and the summary, now match the code.
- **F2:** no remaining text says the removal half removes the bench pairs.
- **F6:** both skip paths warn.
- **F11:** the pattern bullet is present.
- **Renamed helper:** no reference to `resolvePrimeWithoutGit` remains.

Tool calls (4 of 6):
- `grep` for "removal half" and bench-pair removal wording: needed the sites beyond the files already read (F2/F1).
- `grep` for darn writer, bench push and guard wording: needed the other places stating the push admission (F3/F10/F11).
- `grep` for `PrimeWithoutGit`, `pairteardown/doc.go` and `battenshed/deps.go`: needed rename references and the Push/Remove contract text.
- Read `archiveOwnerTip` in `ownerarchive.go`: needed to see whether the tag push honours pushes-off.
