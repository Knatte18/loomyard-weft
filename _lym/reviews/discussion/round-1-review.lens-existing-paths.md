```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced the code paths that already run over each state the record's decisions touch.

**Paths traced, per decision**

- **D2, `lym batten pause`.** It sets batten's own `pause_requested`. Only batten's Run-Shed check (`innerrun.go:782`) and Land read it. The child loom driver, Webster batch verbs and Parent-Align read only the tender pause record (`webstercli/batchpause.go:33-54`, `loomcli/pausehold.go`). **Breaks:** rows, batch commits and Parent-Align keep moving the task branch.
- **D2, creel ask to the writer under a hold.** A pause record entry that covers a strand holds every delivery into its pane (`creelengine/doc.go:258`; `creelcli/delivery.go:373-388`). **Breaks:** the one pause that holds the child also holds the ask.
- **D2, note left uncommitted.** Records commits stage named paths only (`fabricengine/commitweftpaths.go:41`). The `--remove` snapshot keeps the note in a local tag only (`cancel.go:783`). Survives, since the record relies on neither.
- **D4 step 2, `archive_tags`.** `DiscardTaskBranch` hands its remote half to a detached teardown child (`taskbranchdiscard.go:48`, `holdTeardown` handOff at `remoteteardown.go:274-287`, `childHold.finish` at `319-333`). **Breaks:** see finding 2.
- **D4 step 2, records archive.** `RemovePair` with Remote goes through the same hand-off. Its weft tag push is asynchronous and can fail, and the failure is only mailed (`remoteteardown.go:190-207`). **Breaks:** see finding 2.
- **D4 step 2, end-only cancel.** It leaves the run status and returns ok with `end_only` (`cancel.go:237-249`, `266-268`). D4's rule to run the cancel again covers it. Survives.
- **D4 step 3, body edit after the claim clears.** `ClearClaim` writes `draft` (`cancelcmd.go:589-601`), and `checkRunLocks` skips a status that is not a run status (`runlock.go:167`). Survives.
- **D4 step 3, stored record path.** Record-Write reads `ReadStored` before the body (`loomcli/wiring.go:538-559`). `StoreDecisionRecord` writes past the lock and stamps the body sha (`decision.go:47-70`). Survives.
- **D4 step 4, owner after the move.** `Owners.Effective` reads the parent through `ParentOf` (`owners.go:147-163`). `origin.json` is gone after `--remove`, so the board's `project` decides. Survives.
- **D5, same-slug re-run, re-adoption.** `SlugRecordsTags` is reached only from `readopt.go`, during a resume of a live pair whose worktree is gone (`readopt.go:1`, `107`, `149`). A fresh start never re-adopts the old records. Survives.
- **D5, same-slug re-run, tag names.** Archive tag names carry the tip (`archive.go:79`), so the tags of the old and the new cancel never collide. Survives.
- **D5, same-slug re-run, stale tracking ref.** `OriginCodeTip` reads the deleted records branch through its tracking ref. **Breaks** until `stale-records-ref-blocks-rerun` lands; Open risks names it, but no step of D4 runs the prune.
- **D7, refused cancel.** Cancel lifts the slug's tender pauses only on success (`cancel.go:572`). A refused cancel leaves whatever hold the orch set. With batten's pause, `lym batten run` lifts it, so D7 survives as written. Its lift verb no longer matches once the hold changes (finding 1).
- **D7, paused run whose work reached its parent.** The pause help says dobby gives a paused run no pass (`cli.go:387-388`). The orch role says dobby finishes such a run (`orch-role.md:53-54`, `62-63`). Cancel refuses that state first (`cancel.go:330-331`). Survives.

**Findings**

1. **BLOCKING, design.** Location: `_lym/discussion/decision-record.md` D2, first paragraph, with D7's lift.
   - Summary: `lym batten pause` does not hold the inner run. It sets batten's `pause_requested`, which only batten's Run-Shed and Land read, so the inner run's rows go on moving the task branch while the note is written. That breaks the premise that the note's tip SHA and merge base equal what the cancel archives.
   - The hold that does gate loom and Webster is a pause record entry from `lym tender pause <slug>`, and it also holds every creel delivery into the writer's pane, so the D2 ask would never arrive.
   - The record pins neither behavior.
   - Evidence:
     - `internal/battenshed/innerrun.go:400-401`, `782-784`: batten pauses at its own row boundary only.
     - `internal/battencli/cli.go:379-381`.
     - `internal/webstercli/batchpause.go:33-54`: Webster refuses batch verbs only under the pause record.
     - `internal/creelengine/doc.go:258`: "A pause that covers a strand holds every delivery into its pane".
   - Fix: choose a hold that stops the branch movers and still lets the ask through, and state it with its pin. For example, ask the writer first, then hold with `lym tender pause <slug>` and record the tip after the hold. Alternatively, take the tip and merge base from the cancel's archive tag instead of from the note. Then change D7's lift to the verb of the hold chosen.
2. **BLOCKING, design.** Location: `_lym/discussion/decision-record.md` Constraints, the bullet on `--remove`, and D4 step 2.
   - Summary: cancel returns before its archive tags are on origin. In production the task-branch discard and the records archive hand their remote half to a detached teardown child.
   - What `archive_tags` lists:
     - It names only the local tip's tag, before that tag's push.
     - It omits the origin-tip tag when the tips diverge.
     - It is empty when only origin held the branch, though the child still archives origin's tip.
   - A failed push only mails the orch a remote-teardown notice. So "pushes the tags", "when the list is empty, the carrier names no tag" and "the archive tags are on origin" do not hold when the carrier is written.
   - Evidence:
     - `internal/fabricengine/taskbranchdiscard.go:24-41`, `79-97`.
     - `internal/fabricengine/remoteteardown.go:274-287` (`holdTeardown` hand-off), `319-333` (`childHold.finish` returns `handedOff`), `184-189` (failure way forward).
   - Fix: state the asynchronous push and its failure notice. Before step 3, have the orch confirm the tags on origin, for example by listing them with `git ls-remote` on the archive tag pattern, or wait for a missing remote-teardown failure notice. Replace the empty-list rule with that origin check.
3. **MEDIUM, design.** Location: `_lym/discussion/decision-record.md` Open risks, together with D4 and D5.
   - Summary: the existing dobby start path refuses the same-slug re-run while the records tracking ref is stale. The record names the workaround only as an open risk. No D4 step runs it, and no acceptance criterion carries it into `role-beams.md`, so the written procedure stalls at release until `stale-records-ref-blocks-rerun` lands.
   - Evidence: board entry `stale-records-ref-blocks-rerun` body (Cause, Workaround); D4 steps 1 to 4 have no prune.
   - Fix: add the pruning fetch of the records repository to D4, after the cancel and before release, with its evidence, while that entry is unlanded. Or make the release step wait for that entry's landing.

Tool calls:
1. grep for readers of the stored decision record and for add-all records commits: to check whether the note would be swept into a commit and how a fresh run reads a stored record.
2. Read `internal/loomcli/wiring.go:535-584`: to confirm `ReadStored` against `ReadBody` in Record-Write's wiring.
3. grep for the records commit helpers: to confirm records commits stage named paths only.
4. grep for `ArchiveTagPrefix` users: to find every existing path that reads or deletes archive tags on a same-slug re-run.
5. sed of `recordsreads.go:40-80` and `remoteleftover.go:255-285`: to see how `SlugRecordsTags` and `remoteArchiveTags` read the old tags.
6. grep for `SlugRecordsTags` callers: to confirm only re-adoption during a resume reads them, never a fresh start.
