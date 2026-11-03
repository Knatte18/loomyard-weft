```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end on one concrete case and checked the end state against the decision's purpose.

**Findings**

1. **BLOCKING, D7 (it also breaks D8, D9 and Scope).** The guard's test catches the wrong rows.
   - Evidence: `contracts/recipes/darn-recipe.yaml:143-144`, `contracts/recipes/sampler-recipe.yaml:82-83` and `contracts/recipes/tapestry-recipe.yaml:468-469` (Code-Bouncer) all list `_lym/discussion/decision-record.md` under `artifact_paths`. Only `tapestry-recipe.yaml:140-141` is the Discussion-Bouncer.
   - Walkthrough: the guard as written fails on each Code-Bouncer row. To make it pass, an implementer would add `approve_seam: discussion` to darn's, sampler's and tapestry's Code-Bouncer rows.
   - Result: every Code-Review approval would store the record, including on darn and sampler runs, and flag those entries discussed. The past-approval check in D8 and D9 would then count darn and sampler as Discussion recipes. That contradicts D8 ("a darn run, a sampler run … never copied"), D9's darn refusal and the Scope line "darn unchanged".
   - Fix: key the guard on rows of a Discussion review only. That means the Bouncer of the segment that follows a `DiscussionWrite` row, or a Bouncer whose `role_prefix`/`run_subdir` is discussion. It must not key on `artifact_paths`.
   - Second walkthrough, same root: the Code-Bouncer's settle happens after Discussion-Bouncer. If the seam sat on the Code-Bouncer, it would overwrite the board copy with the record as it stands at Code-Review, carry-overs included.

2. **LOW, D6 ("runs after the segment's carry-over").** That holds only on the judged-settle path.
   - Walkthrough: with `discussion_max_bounces` 0 the Bouncer goes through `approveWithoutReview`, which calls `approveAndCommit` directly with no CarryOver.
   - The end state is still right: the stored bytes equal the committed bytes, so the purpose holds. The sentence just overstates the ordering.

**Per decision: the case I ran, and whether the purpose was met**

- **D1:** stored a 20 KB record for T2. The file is written at the Bolt root, `board.json` gets `discussed:true`, and the render manifest never lists the file, so a re-render keeps it. Met.
- **D2:** stored on T2 while it is held, status `running · Discussion-Bouncer`. Only `discussed` changes, `scopeChanged` ignores it, and the revised date is not stamped. A repeat with the same bytes is a noWrite, so no sync is spawned. A crash after the file write but before the save leaves a file without a flag, and `DecisionRecord` reports not-found. Met.
- **D3:** `get` → edit body → upsert with `discussed:true` on a discussed entry is a no-op and passes. `discussed:true` on an undiscussed entry is refused. Met.
- **D4:** rename T2 → x moves the file. Prune of done T2 removes the file in afterSave. `batten cancel` writes only the status, so the flag and file stay. Met.
- **D5:** machine A stores, origin is untouched, the replay writes the file, and CommitRuns names T2. If origin removed T2, the file is reported lost. Met.
- **D6:** a tapestry judged approval goes CarryOver (commits) → Approve (reads the working copy, stores it under the slug from `ResolveRunID`, which is the same slug `seedSlug` gives) → CommitDiscussion lands the same bytes. Met, with finding 2.
- **D7:** broken, see finding 1.
- **D8:** `decision add` on tapestry at Webster: `rowWritten` is false, the commit happens, the current row index is after the Discussion-Review rows, so the committed bytes are copied. At Discussion-Burler-Fix there is no copy. Darn has no discussion seam row only if D7 is fixed. Met once D7 is fixed.
- **D9:** an in-flight run past approval when the new binary lands (the embedded recipe now has the seam) is copied. Met, under the same D7 caveat.
- **D10:** `get --record` reads through `DecisionRecord`; `--body` beside it is refused. Met.

**Tool calls**

- grep of `artifact_paths` and `approve_seam` across `contracts/recipes/*.yaml`: my context had only tapestry's Discussion-Bouncer and Plan-Bouncer configs, not which other Bouncer rows name the decision record.
- grep of `ResolveRunID` and `seedSlug`: to confirm the board slug D6 uses matches the one D8 and D9 use; my context had only the call sites.
