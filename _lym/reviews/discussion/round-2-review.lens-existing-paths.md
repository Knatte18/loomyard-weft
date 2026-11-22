```yaml
lens: existing-paths
outcome: reported
```

My lens is existing-paths: for each state the decisions create or change, I traced the code paths that also write or read that state, and checked whether the decision's result survives them.

**Paths traced, per decision**

- **D1: board accepts the edit only after cancel sets the entry to `draft`.** `checkRunLocks` skips any status that is not a run status (`internal/boardengine/runlock.go:166-168`). The pair check passes once `--remove` has run (`project.go:35-36`). Survives.
- **D2: ask before the hold.** The ask arrives because creel holds delivery only under a pause that covers the strand (`creelengine/doc.go:258`). Survives.
- **D2: the writer writes `_lym/move-note.md`.** The Discussion writer's stencil makes every file but its three outputs read-only (`loom-template-discussion.md:156-172`). The parent directive tells a session to refuse an instruction outside its role's limits and say so to the sender. Breaks.
- **D2: the hold is `lym tender pause <slug>`.** With a slug, the pause is also written as the run's pause intent in batten's record (`tendercli/cli.go:114`). Cancel lifts the slug's own pause entries (`cancel.go:572-576`), and `--remove` deletes batten's record. Survives.
- **D2: the note stays uncommitted under `--remove`.** `RemovePair` runs with `Snapshot: true` (`cancel.go:783`), so the note also lands in the local snapshot tag. Survives.
- **D4 step 2: parent check, done or held by a run.** It matches `checkProjectField` (`project.go:97-100`) and `RunHolds` (`runlock.go:56-62`). Survives.
- **D4 step 2: "`lym beam ready` has run", read with `lym board get`.** The ready flag is a record committed in the beam worktree's `_lym/landing` (`beamcli/cli.go:916-921`), not a board field, so `lym board get` cannot show it. Breaks.
- **D4 step 3: partial finish versus refusal, judged from the cancel's output.**
  - The `end_only` and `way_forward` envelope keys exist (`cancelcmd.go:171-172`).
  - The hub-pause refusal reads `run "lym tender resume", then "lym batten cancel <slug> --remove" again` (`cancel.go:303-304`). That names the cancel again, so D4 files it as a partial finish, and the orch "takes the fix it names". The fix lifts a hub pause the operator set, before anything was ended.
  - The owned-elsewhere refusal names `"lym batten cancel <slug>" on <owner>` beside `lym adopt` (`cancel.go:715`). It matches both of D4's classes.
  - Breaks.
- **D4 step 3: tag name taken from the recorded tip.** It matches `ArchiveTagName` (`archive.go:78-79`) and the containment rule of `archiveDiscardedTips` (`taskbranchdiscard.go:104-134`). Survives.
- **D4 step 4: discussed entry stored with `--record-file` right after step 3.**
  - `UpsertTaskWithRecord` refuses while the slug's weft branch is live (`recordupsert.go:38-46`).
  - `RecordsBranchLive` reads origin's records branch as live until an archive tag covers it or the teardown child deletes it (`remoteleftover.go:86-93`, `:111-113`).
  - Step 3 waits only for the code tag. The records-branch wait is in step 5, after the upsert.
  - Breaks.
- **D4 step 4: one upsert sets both `project` and the record.** `UpsertTaskWithRecord` takes the same fields. `judgeRecordReplaceIn` passes once the entry is a draft with no pair (`recordupsert.go:101-127`). Survives.
- **D4 step 5: pruning fetch, then release.** Survives, per the stale-records entry's body.
- **D5: owner read through the parent.** `OwnerEntry.Owner` is recorded only for beam projects (`owners.go:24-25`), and PATTERN-owner-machine background line 30 agrees. Survives.
- **D6 and the owner Constraint: the fresh run on the new owner reads the code archive tag.**
  - The code store fetches with plain `fetch` and default refspecs (`gitrepo/pull.go:39`, `fabricengine/originreconcile.go:569`).
  - By git's documented default, `git fetch` follows only tags that point into fetched history. An archive tag of a deleted branch is outside it, so it never reaches the new machine.
  - The record has no step that fetches the tag, yet it claims the run "needs nothing local to the old machine".
  - Breaks.
- **D7: lift with `lym tender resume <slug>`.** It matches the D2 hold. Resume with a slug lifts that pair's own entry (`tendercli/cli.go:194-199`). Survives.

**Findings**

1. **MEDIUM, design. `_lym/discussion/decision-record.md:99-103` (D4 step 4, discussed-entry branch).**
   - Summary: the `--record-file` upsert runs before the slug's records branch is gone from origin. The board refuses it while the weft branch reads live. Step 3 waits only for the code tag, and step 5's records-branch wait comes after the upsert.
   - Evidence: `internal/boardengine/recordupsert.go:38-46`, `internal/fabricengine/remoteleftover.go:86-93`, `:111-113`.
   - Fix: move the wait until `git -C <records repo> ls-remote origin` lists no records branch of the slug ahead of step 4.

2. **MEDIUM, consistency. `_lym/discussion/decision-record.md:90-94` (D4 step 3 classification).**
   - Summary: "an error whose way forward names the cancel itself to run again" is not a partial-finish test.
     - The hub-pause refusal, made before anything ended, names `lym tender resume` and then the cancel again. The orch would lift the operator's hub pause as the "fix".
     - The owned-elsewhere refusal names the cancel on the owner machine as well as `lym adopt`, so it matches both classes.
   - Evidence: `internal/battencli/cancel.go:303-304`, `:715`.

3. **MEDIUM, design. `_lym/discussion/decision-record.md:86` (D4 step 2, ready-beam bullet).**
   - Summary: the check runs through `lym board get`, but `lym beam ready` writes its flag as a record in the beam worktree's `_lym/landing`, not on the board. The rule names a state its own tool cannot read.
   - Evidence: `internal/beamcli/cli.go:916-921`.

4. **MEDIUM, design. `_lym/discussion/decision-record.md:42-43` (D2 ask).**
   - Summary: the live writer is asked to write `_lym/move-note.md`, outside its stencil's write surface. The Discussion writer may write only its three files, and the parent directive has it refuse an instruction outside its role's limits. D2 has no path for a refusal, and D3 covers only a writer that is not live.
   - Evidence: `_bolt/_lym/stencils/loom/loom-template-discussion.md:156-172`.

5. **MEDIUM, design. `_lym/discussion/decision-record.md:166-167` (owner Constraint) and `:129-130` (D6).**
   - Summary: the fresh run on the new owner machine reads "the diff from the merge base to the tag". The code store's plain fetch never brings an archive tag whose branch was deleted, since auto-follow covers only tags into fetched history. No step fetches `refs/tags/archive/<slug>/<hex>`, so the claim "needs nothing local to the old machine" fails on another machine.
   - Evidence: `internal/gitrepo/pull.go:39`, `internal/fabricengine/originreconcile.go:569`, and git's documented tag auto-following.

**Tool calls**
1. A grep of `"fetch"` and tag-mode use in `internal/fabricengine` and `internal/gitrepo`. My context did not show how the code store fetches, which finding 5 needed.
