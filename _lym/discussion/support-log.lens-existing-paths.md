```yaml
lens: existing-paths
outcome: reported
```

Lens existing-paths: I traced the paths that already write or read what D1–D10 add. One finding breaks D7, two break D5 edge cases, and the rest survive.

**Findings**

1. **BLOCKING — D7, and through it D8 and D9.** The guard's selector also catches review segments that are not Discussion's.
   - The record says the guard covers "every Bouncer row whose `artifact_paths` name the discussion decision record".
   - That record path appears in `artifact_paths` on darn's Code-Bouncer (`contracts/recipes/darn-recipe.yaml:143-144`), sampler's Code-Bouncer (`contracts/recipes/sampler-recipe.yaml:82-83`) and tapestry's Code-Bouncer (`contracts/recipes/tapestry-recipe.yaml:468-469`). It also appears on tapestry's Discussion-Bouncer.
   - As written, the guard would require `approve_seam: discussion` on all three Code-Bouncers. Approving code review in darn, sampler and tapestry would then store darn's body-copy record and flag the entry discussed.
   - D8 and D9 decide whether a run "has a Bouncer row with `approve_seam: discussion`", so they would inherit the error. That contradicts their own claim that darn and sampler are never copied.
   - Fix: select the review segment entered from a `DiscussionWrite`-engine row. That is the Bouncer named by the DiscussionWrite row's `on_done`, together with its segment.

2. **MEDIUM — D5.** A replayed removal of an entry and its record file leaves an orphan file on origin.
   - D4 removes the record file in the same write that removes the entry, so one commit carries both.
   - On replay the entry removal is applied first. D5's rule then reports lost "a record file whose entry origin's board does not hold after the entry changes", and the commit's file removal (After nil) falls under it.
   - So origin keeps `decision-&lt;slug&gt;.md` with no entry. That is the orphan D4 exists to prevent, and a later entry reusing the slug inherits it, although `DecisionRecord` hides it until a store.
   - Fix: apply the entry check only when the commit writes the file. A removal follows the design-file rule.

3. **LOW — D2 and D9.** A store on a done entry may never reach origin.
   - The run ledger sets aside a commit whose runs have all ended when it touches no other replayer's file (`internal/fabricengine/replay.go:675-695`). `CommitRuns` names the record file's slug (D5).
   - A store whose sync commit carries only that record file is therefore set aside rather than replayed when the entry is done.
   - This is reachable only by D9 after a landing. The record should say so: allow D9 only on an open entry, or accept the case.

4. **LOW — D1, D4 and D5 with an older binary.** Mixed binaries are not bounded.
   - An older binary keeps `discussed` as an unknown key, but its prune, remove and rename leave `decision-&lt;slug&gt;.md` behind.
   - Its replayer does not claim `decision-*.md`, so a commit holding the file takes the non-by-entry path (`replay.go:675`).
   - The record states no bound for this. One line under Auto-mode assumptions, like the board doc's migration bounds, would cover it.

**Paths traced**

| Decision | Path | Result |
|---|---|---|
| D1 | Render manifest removal in `RenderToDisk`: deletes only names from the previous manifest | survives |
| D1 | Bolt `.gitignore` patterns (`*.lock`, `*.swaplock`, render manifest): the record file is committed by board sync | survives |
| D1 | Unclaimed-path dispatch in fabric replay (`claimantOf`) | survives once D5 claims the file |
| D2 | `checkRunLocks` and `scopeChanged`: `discussed` is excluded; `stampDates` follows scope | survives |
| D2 | Replay's run-lock check on a held entry: `discussed` is not scope | survives |
| D2 | Failed critical section after the in-mutation file write: file left with no sync until the next board write | survives (stated) |
| D3 | `UpsertTask`, `UpsertTasksBatch`, `MergeTasks` payload checks, as for `beam` and `front` | survives |
| D4 | `lym batten cancel` writes only `SetStatus` (`internal/battencli/cancelcmd.go:325`) | survives |
| D4 | `Rename`, `Prune`, `MergeTasks` removal, `RemoveTask` | survive as specified |
| D4 | Older-binary prune, remove and rename | break (finding 4) |
| D5 | `CommitRuns`, `Claims` and the entrymerge field table | survive |
| D5 | Replayed removal of a record file | breaks (finding 2) |
| D5 | Ended-run set-aside in the run ledger | edge case (finding 3) |
| D6 | Every approving Bouncer path calls `approveAndCommit` (`bouncer.go:522, 746, 1167, 1279`) | survives |
| D6 | Re-settle over an unchanged subject (`bouncer.go:833`): re-stores the working record, idempotent | survives |
| D6 | Overlay fixer round (no git): the seam reads the working copy | survives |
| D6 | Carry-over from later Plan and Code Bouncers into the record: board copy stale until the next add or store | survives (stated in Auto-mode assumptions) |
| D7 | `artifact_paths` across recipes | breaks (finding 1) |
| D8 | `--amends-plan` path: the copy runs after commit and after `afterFailedCommit` lands | survives |
| D8 | Run sent back into Discussion: the row-order check holds | survives |
| D8 | Run halted at Discussion-Bouncer | survives (stated) |
| D8 | "Recipe has an `approve_seam: discussion` Bouncer" | inherits finding 1 |
| D9 | Store on a landed entry | edge case (finding 3) |
| D10 | `get --body` shape; `discussed` is omitted when false | survives |

**Tool calls**
- `internal/fabricengine/boltreplay.go` and a grep of `Claims(`: how fabric treats Bolt paths no replayer claims. My context did not have fabric's replay dispatch.
- `internal/boardengine/sync.go` `.gitignore` patterns and the board writes in `internal/battencli/cancelcmd.go`: whether a record file is committed, and what cancel writes. My context did not have either.
- `internal/fabricengine/boltreplay.go:120-175` and a grep for unchanged-subject re-entry in `bouncer.go`: the replayer lookup and the Bouncer re-entry path, which my context had not seen.
- `grep claimantOf` in fabricengine: how a commit's paths decide by-entry replay and the ended-run set-aside. My context did not have this.
- `grep artifact_paths` across `contracts/recipes/*.yaml`: which Bouncer rows name the decision record, needed to test D7's selector. My context had only tapestry's Discussion-Bouncer row.
