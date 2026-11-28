```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. I traced each function and branch the diff adds or changes.

**Per function or branch: the case I ran, and whether it is correct**

- `shedbuild.expandRows` / `checkRunEndingBlock`:
  - tapestry's top level, `Jacquard` at index 1 of 2, empty `on_done`, gate `always`: blockExit is "", `last` is true and the gate is `always`, so it is admitted. The inlined rows get exit "", so Friction-Reflect keeps an empty `on_done`. The emitted block row has `SkipTo` "", which skips by `OnDone`, but `always` never skips. Correct.
  - A block with empty `on_done` that is not last at the top level: refused with the old message. Correct.
  - Last, gate `undiscussed`: refused with the new message. Correct.
  - A nested block inside a run-ending block, with empty `on_done` and not last in its sub-recipe: the enclosing exit is "", so it is refused. Nested and last with gate `always`: admitted. This matches the summary's claim.
  - A nested block whose enclosing exit is non-empty: takes that exit, unchanged from before. Correct.
- `shedbuild.recordSourceRow`:
  - 0 record rows and 0 seed rows: refused. 2 record rows: refused. 2 seed rows: refused.
  - Seed rows only: returns the seed row. Record rows only: returns the record row.
  - Both, seed after record on the path (expanded tapestry: Discussion-Write … Discussion-Bouncer → Jacquard → Preflight → Record-Seed): returns the record row. Correct.
  - Both, seed ahead of record: `seedAt < recordAt`, refused. Correct.
  - Both, seed off the path: `seedAt` is -1, refused. Correct.
  - Both, record off the path: `recordAt` is -1, refused. The message says "seed does not lie after record", which is acceptable. Correct.
- `checkLoomContract`, seed-only with a build row (jacquard): the seed row is Preflight's `on_done`, ahead of Webster, so the path check passes. A seed row off the path is caught by the build-order clause. Correct.
- `rowsOf` / `onlyRowOf` refactor: same behaviour as before. Correct.
- `loomrecipe.RecordEngine`:
  - tapestry: Discussion-Write comes first, returns DiscussionWrite. jacquard: returns RecordSeed. darn: returns RecordWrite. A recipe with neither: "".
  - A seed row ahead of a record row in row order: the record row is still returned. Correct.
- `recordWrite.Call` in stored-only mode (`noStoredReason` set):
  - Record missing, stored record found: writes the stored record and the stored-record support log, commits, Done. Correct.
  - Record missing, no stored record: computes the reason, checks cancel, returns Stuck. Nothing is written, the directory is not created and there is no commit. Correct.
  - Record missing, no stored record, reason seam fails: `p.fail` wraps with `%w`, or returns the cancel error when cancelled. Correct.
  - `ReadStored` fails: `p.fail`, nothing written. Correct.
  - Record exists, support log exists: no stored read, commit, Done. Correct.
  - Record exists, support log missing, no stored record: writes `boardSupportLog()`. That log says the record "was written from the board entry, with no interview", which is false in this mode (finding 1).
  - Commit fails: returned error, the files are left written, and a re-invoke completes it. Correct, as for Record-Write.
- `recordSeedStuckReason`:
  - `DiscussionRow` "Discussion-Write": `ReadSeed` is not called, and the reason names goto then resume. Correct.
  - `DiscussionRow` "": reads the seed and renders the batten and re-seed sequences.
    - The `lym shed seed self --recipe --driver --param` flags exist.
    - Under the re-seeded tapestry, `lym loom goto --to Discussion-Write` moves the run back from Record-Seed (step 6 to step 2), and Discussion-Write is not a review row, so goto allows it.
    - The seed keeps the same driver and parent, so the start does not hit the disagreeing-seed refusal.
    - Correct.
  - `ReadSeed` fails: the error is propagated. Correct.
- `loomcli.recordSeedDeps`:
  - `c.runID` is set in `resolveRunID` (`arm.go:72`) before `wire`.
  - `seedSlug` is the worktree base name, never empty, so `recordSeedEntry` does not refuse at build time.
  - `ReadSeed` with no seed would render an empty driver and parent. This is unreachable: the generic verbs refuse without a seed (`arm.go` `resolveRunID`), and start writes the seed first. Correct.
- `loomcli.wire`, the `EngineRow(c.recipe, DiscussionWrite)` call: tapestry returns Discussion-Write, jacquard "", arras Discussion-Write through the expanded Motif. It fails only where `RecordEngine` already would. Correct.
- `shedrecipe.recordSeedEntry`: rejects unknown config keys, a relative path, an empty slug and each nil seam. A filled Env builds. Correct.

**Findings**

1. LOW · design · `internal/loomshed/recordwrite.go:211-215`
   - **Summary:** in Record-Seed's stored-only mode, an existing record with a missing support log and no stored record gets the board-rendered support log. That log claims the record "was written from the board entry, with no interview", which is false for a seed row, since the seed row never renders from the body.
   - **Evidence:** `recordwrite.go:113-121` (`boardSupportLog` text), `recordwrite.go:211-215`, `recordseed.go:133-145`.
   - **Fix:** in stored-only mode, write a support log that does not claim a board-rendered record, or leave the support log alone when no stored record exists.

Tool calls:
1. `seedinput.go` `seedSlug`, `cli.go` runID, `SeedFile`: to check whether `RecordSeed.Slug` or `SeedPath` could be empty or relative at wire time.
2. `arm.go` `c.runID` assignment, `SeedFile` in `paths.go`, `lymcwd` `WorktreeName`: to confirm `runID` is set before `wire` and the slug is non-empty.
3. `arm.go:55-80` `resolveRunID` and the `.wire(` call sites: to confirm the order of the runID resolution and `wire`.
