```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker lens. I tried to break each claim and bound the decision record states.

Claims attacked:
1. D2: "`lym batten pause` stops every row, batch commit and Parent-Align merge-in" — **broken**.
2. D2: "the live writer can be reached by creel while the run is held" — **broken** once the run is held by the only pause that does stop the inner run.
3. D2: "the pause keeps the note's tip SHA and merge base equal to what the cancel archives" — **broken**, because it follows from claim 1.
4. D7: "lifts its pause with `lym batten run <slug>`" — **broken** for a pause that actually holds the inner run.
5. Constraint: "With `--remove`, cancel archives each branch tip … pushes the tags" — **broken** in production, because the push is asynchronous and can fail.
6. Constraint: "`archive_tags` lists the code branch tip's tags" and D4 step 2 "an empty list means no tag" — **broken**.
7. Constraint: "the archive tags are on origin, so the fresh run needs nothing local" — **broken** in the window after cancel returns, and after a failed child push.
8. D1 bound: "the board accepts the edit only because cancel set the entry to `draft`" — held for a body edit. It is overstated for the `--record-file` path, which bypasses the run lock anyway (NIT, not filed).
9. Constraint: "Cancel sets a run-form status back to `draft`" — held (`cancelcmd.go:589-601`). The exception is an end-only cancel, which D4 step 2's re-run covers.
10. Constraint: "Cancel without `--remove` makes no archive tags and refuses a re-run" — held (`cancel.go:539-561`, `cancel.go:340-342`).
11. Constraint: "a task's owner is read through its parent" — held (`owners.go:141-163`).
12. Constraint: "cancel refuses on a non-owner machine" — held (`cancel.go:625-631`, `cancel.go:642-643`).
13. Constraint: "an entry with a stored record runs from it, never the body" — held (`decision.go:90-98`, `decision.go:112-125`).
14. D7: "cancel's refusals guard the states where a move would lose landed work" — held for a task (`cancel.go:326-332`).

Findings:

- **BLOCKING · design** · `decision-record.md` D2 (and D4 step 1, D7)
  `lym batten pause` holds only batten's own outer run, never the inner loom or Webster run that moves the task branch.
  - The pause sets `pause_requested` on batten's status. The Run-Shed producer reads it only to end its own wait and pause batten at its row boundary.
  - The child driver in the task worktree never reads that flag. Loom (`internal/loomcli/pausehold.go`) and Webster (`internal/webstercli/batchpause.go:33-54`) honour only `pauserecord` coverage, which is the `lym tender pause` record.
  - Counterexample: during Webster, the orch runs `lym batten pause`. Merriam keeps beginning batches and committing, and the next Parent-Align merges the parent in. The tip SHA and merge base in the note drift from what cancel archives.
  - The orch role text itself splits the two holds: `orch-role.md:60-63`.
  - Evidence: `internal/battenshed/innerrun.go:400-401` and `innerrun.go:782-783`; `internal/battencli/cli.go:379-381`.

- **BLOCKING · design** · `decision-record.md` D2
  The only hold that does stop the inner run, a `lym tender pause <slug>` entry, also stops creel delivery to the writer's strand.
  - Counterexample: the orch tender-pauses the slug, then sends `lym creel post --expect-reply <writer-strand>`. Creel withholds the delivery while the pause covers the strand, so the writer never gets the ask.
  - D2 rests on holding the run and reaching the writer at once. Neither existing hold gives both, and the record pins neither behaviour.
  - Evidence: `internal/creelengine/doc.go:258` ("A pause that covers a strand holds every delivery into its pane"); `internal/creelengine/deliver.go:386`; `internal/creelcli/delivery.go:373-388`.

- **MEDIUM · design** · `decision-record.md` D7
  "lifts its pause with `lym batten run <slug>`" lifts batten's `pause_requested` only.
  - Once the hold is fixed to a `lym tender pause` entry, the lift is `lym tender resume <slug>`, and a refused cancel leaves that entry standing.
  - `lym batten run` itself refuses while a tender pause covers the slug.
  - Evidence: `internal/battencli/pausehold_test.go:27-28`, `pausehold_test.go:58` (the refusal names `lym tender resume`); `cli.go:328-329`.

- **BLOCKING · design** · `decision-record.md` Constraints (the `--remove` bullet) and the owner-machine bullet's "the archive tags are on origin"
  `DiscardTaskBranch` hands its remote half to a detached teardown child outside test binaries and returns without waiting.
  - At the moment cancel returns, the code tag in `archive_tags` exists locally only. Its push to origin runs later in the child.
  - A failed push is only logged, with a mailed orch notice naming a manual `git push` of the tag.
  - Counterexample: origin rejects the push. The carrier names `archive/<slug>/<hex>`, the fresh run starts on the new parent's owner machine, and no such tag exists on origin.
  - The record cites `cancel.go`/`cancelcmd.go` for this behaviour, but it lives one hop further, in fabricengine, and is unpinned there.
  - Evidence: `internal/fabricengine/taskbranchdiscard.go:30-37` and `taskbranchdiscard.go:91-97`; `internal/fabricengine/remoteteardown.go:274-286` and `remoteteardown.go:319-332`; `remoteteardown.go:184-189` (the failure notice).

- **MEDIUM · design** · `decision-record.md` D4 step 2 and Constraints ("`archive_tags` lists the code branch tip's tags"; "when the list is empty, the carrier names no tag")
  On the hand-off path, `ArchiveTags` holds only the local tip's tag, which leaves two cases wrong:
  - **Diverged tips.** When the local and origin tips diverge, the child pushes a second tag for origin's tip that is never reported.
  - **Origin-only branch.** When only origin holds the branch (no local tip), `archive_tags` is empty, yet the child archives origin's tip under `archive/<slug>/…`. The carrier then names no tag, and the code that exists is dropped.
  - Evidence: `taskbranchdiscard.go:79-89` (local tag only), `taskbranchdiscard.go:93-96` (in-process path only); `taskbranchdiscard.go:105-134` (`archiveDiscardedTips`, two tags or an origin-only tag).

- **LOW · design** · `decision-record.md` D4 step 3 and Constraints (the records archive tag "in the records repository")
  The records tag is pushed to origin under the store's tag namespace (`ns.Tag(tag)`), not as `refs/tags/archive/<slug>/…`.
  - On a one-repository hub, a reader on the new owner machine finds it on origin only through the records store's refspec mapping.
  - "pattern `archive/<slug>/` in the records repository" is true of the local repository on the old machine only.
  - Evidence: `internal/fabricengine/archive.go:224-238`; `internal/gitrepo/refnamespace.go:28-31`; `remoteteardown.go:200-203` (the manual way forward spells the namespaced destination).

No tool calls made; every claim above was judged from the inherited context.
