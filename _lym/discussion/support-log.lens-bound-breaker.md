```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked the record's bounds, guarantees and claims about existing code. Six findings: one BLOCKING, four MEDIUM, one LOW.

**Findings**

1. **BLOCKING — D6, a failed filing deletes the arras entry's stored record.**
   - Counterexample: in the family case the project slug is the run's own slug, so `fileRaddleEntry` writes `decision-<arras-slug>.md` and appends it to `recordFiles`. That file already holds the record the Motif `approve_seam` stored.
   - If the board write then fails before the save, `FileRaddle` removes every path in `recordFiles` with `os.Remove`. A validateWrite refusal of the in-place kind change, or a re-check refusal inside the write, is enough.
   - The entry keeps `Discussed: true` with no record file. The next Raddle-Write, or a restart through the `undiscussed` gate, finds no stored record.
   - Fix: D6 must keep the run entry's existing record out of the rollback, or restore it.

2. **MEDIUM — D8, "a project's own run lock refuses the write as it does today" is false.**
   - Status is not a scope field: `EndRaddleEntry`'s comment says "the run lock admits the write since neither field is scope". `RunHolds` is also false for a beam project at the hold-free rows (`Worktree-Create`, `Preflight`, `Record-Write`, `Beam-Up`, `Beam-Await`).
   - Counterexample 1: `set-status null` on a beam project at Beam-Await clears the pawl run's `running · Beam-Await` status and releases its draft members.
   - Counterexample 2: a done project that prune keeps because members survive is reopened by a null status.
   - The bound must name these cases: refuse a project whose status is a run status or `done`, or release only the members and leave the project's own status alone.

3. **MEDIUM — D5 and D6, entries that depend on the arras entry.**
   - The board allows `depends_on` on a task or a project (`store.go` validateWrite refuses only notes and isolated tasks).
   - One-task outcome: `EndRaddleEntry` marks the arras entry done. A dependent X becomes ready at once, before the filed task has run, so X builds on work that has not landed.
   - Family outcome: X ends up depending on the project, and the record says nothing about when the project counts as done.
   - The record is silent on both. It should state the rule, for example moving dependents' `depends_on` onto the filed task or tasks in the filing write, or naming the behaviour as accepted.

4. **MEDIUM — D4 and D2, a Raddle-File return may skip the perch.**
   - After a return ("board moved", an invalid proposal or a hash mismatch), Raddle-Write writes a new proposal and the Bouncer is entered again.
   - The Bouncer has a "re-entry over a settled verdict" path (`internal/shedadapters/bouncer_reentry_test.go`) that re-settles from its approved settle record without judging.
   - The new proposal could reach Raddle-File unreviewed. That breaks D4's claim that a run files on the perch's approval.
   - The record's plan note covers only whether the segment layout makes the return legal. The record should require that a return re-arms the perch (clears its settle record) so the new proposal is reviewed.

5. **MEDIUM — D6, `SetRunStatus` and the end-of-run writes.**
   - Recognising "a project filed from a raddle run under its own slug" works by body prefix (`isRaddleFiledFrom`).
   - The conversion keeps the former body after the prefix, so the prefix test holds. But any later operator edit of the project body that drops the prefix (body edits are allowed once the run no longer holds the entry) silently removes the guard.
   - More concretely, the batten rows after Raddle-File still write `running · Land` and similar statuses through `battencli/commitstatus.go`, and the record names only `SetRunStatus`. Each writer must apply the skip, or the guard must key on the run's own state, not on the entry's body.

6. **LOW — D6, "the run lock admits this write only from Raddle-File".**
   - The board cannot tell callers apart. `endsAsSketch`, which D6 mirrors, judges only the before and after shapes of the entry.
   - Any writer that produces the same change passes, including an orch `lym board upsert` that sets the kind to project with the prefix body.
   - The bound should be stated as a shape-based admission, not a caller-based one.

**Claims attacked**

| Decision | Claim | Result |
|---|---|---|
| D1 | One record row: no Record-Write, the Motif block's DiscussionWrite is the only one | held |
| D1 | `finishesAsSketch` is false because of the top-level RaddleFile row | held |
| D1 | A restart after approval resumes at Raddle-Write through the `undiscussed` gate | held, but breaks after finding 1's failed filing |
| D2 | Decision-class findings reach the parent through the arbiter | held (from exploration) |
| D2, D4 | A Raddle-File return is reviewed again | broken (finding 4) |
| D4 | Raddle-File still runs every check | held |
| D5 | The one-task outcome's done entry is pruned at landing | held: `battenshed/land.go` prunes on the landed-no-code path too |
| D5 | `slug-taken` and `slug-sketch` can exempt the project slug | held: the run entry is already exempt from `slug-taken` |
| D5, D6 | Entries depending on the arras entry | broken (finding 3) |
| D6 | The retry finds the converted entry as filed | held: body prefix plus `Discussed` |
| D6 | Rollback leaves the stored record intact | broken (finding 1) |
| D6 | Only Raddle-File can make the conversion | broken (finding 6) |
| D6 | Run-status writes skip the converted project | partly held (finding 5) |
| D7 | `decision add` past approval stores the record | held, since `pastApproval` is decided by rows |
| D7 | Every part carries the post-Discussion entries | held (`SplicePart`) |
| D8 | The run lock refuses a project release | broken (finding 2) |
| D8 | Dobby orders released members by `depends_on` | held (`PendingDependencies`) |

**Tool calls**

1. Grepped `PruneBoard` in `internal/battenshed`, to see whether the landing prunes a run that landed no code; my context did not have it.
2. Read `battenshed/land.go` lines 185–245, `runlock.go`, and the depends-on checks in `validateWrite`, for prune on the no-code path, run-lock coverage of status and projects, and whether a task may depend on a project; not in my context.
3. Listed the shedadapters Bouncer files and grepped for settled or re-entry handling, to see Bouncer re-entry after an approved settle; not in my context.
4. Read the header of `bouncer_reentry_test.go`, to confirm the re-entry-over-a-settled-verdict path exists; not in my context.
