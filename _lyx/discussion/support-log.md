# Support log — orch-rules-files

## Interview

### Turn 1 — exploration (autonomous)

Read the board entry and the related entries `orch-any-worktree`, `hub-workspaces`, `pair-from-parent`, `umbrella-daemon`, `orch-split` and `umbrella-anchor`.
Read how `ORCHESTRATOR.md` reaches the role file (`orchPaths`, `RenderRoleFile`, `readRepositoryRules`, the role stencil's closing section), the Bolt commit route `configcli` uses for hub config, and the hub layout on disk.
Found: the prime's `_lyx` is a link to `loomyard-weft/_lyx` on `main-weft`;
a pair's weft forks from `main-weft`;
weft content never merges at landing (`fabricengine` doc, Durable-vs-Ephemeral State);
Bolt files are committed only through `fabricengine.Bolt` (`CommitWritten`, `Push`).

### Turn 2 — design questions, settled by auto-pick

1. Where the current rules go. Recommended: both to the hub-wide file, since both hold for any orch of the hub. Picked.
2. How the hub-wide file is committed. Recommended: a `lyx orch rules --from <file>` verb that writes under the board write lock and commits one path. Alternatives: in-place edit then a commit verb; no verb. Picked the recommendation.
3. How the own file is committed. Recommended: the existing `lyx fabric commit`/`sync`, since it is a `main-weft` file in the prime. Picked.
4. Render shape. Recommended: two closing H2 sections, hub-wide first, each with a placeholder naming its path. Alternative: one section with two subsections. Picked the recommendation.
5. Who moves the content. Recommended: `ly:orch` after landing and deploy, since no card can write Bolt or `main-weft`. Picked.

### Turn 3 — lens review

Every lens returned a report.

- premises, scenarios, pattern-fit (merged): D3 rolled back after a failed push, though `setHubWide` restores only inside its write closure — fixed: restore on write or commit failure only, a failed push is reported as not pushed, with a criterion.
- premises: D3's dirty-Bolt rationale was speculative — fixed: restated as `Bolt.Commit` sweeping every change into an unrelated commit.
- premises: the orch pre-run resolves engine config for every verb — fixed: plan-writer note.
- premises: "unchanged content commits nothing" is unconfirmed for a single path — fixed: plan-writer note; the criterion stays.
- premises: own file committed by `lyx fabric commit`/`sync` is unverified — rejected: the prime's `_lyx` is the `main-weft` worktree and the index says those verbs commit weft changes.
- scenarios: an old watcher refuses renders against the reseeded stencil — fixed: D4 step 1 restarts the orch, and the open risk names it.
- scenarios: first write with no `_lyx/orch/` on Bolt — fixed: D3 creates the directory, with a criterion.
- scenarios: empty `--from` unspecified — fixed: accepted as the way to clear, with a criterion.
- scenarios: non-atomic write — fixed: temp file and rename in D3.
- scenarios: relative `--from` resolution — fixed: resolves against the caller's working directory.
- scenarios: `--from` publishes any readable file — fixed: stated in D3's bound.
- scenarios: no `resume-context` criterion — fixed: criterion for `start`, the watcher and `resume-context`.
- board-fit: D3's bound rests on the prime-only refusal `orch-any-worktree` removes — fixed: open risk naming that `orch-any-worktree` must re-bound the verb.
- board-fit: weft fork source wording ignores `pair-from-parent` — fixed: the open risk says "parent worktree's weft" and names `pair-from-parent`.
- board-fit: synchronous push differs from `batten-start-speed`'s async per-transition pushes — rejected: an operator-run verb, not a per-transition push.
- board-fit: `orch-split` not named — rejected: the anchor-derived own file needs nothing from it.
- goal-scope, leanness (merged): the move is a manual orch step with no route to the orch and no verbatim text — fixed: D4 gives the exact text and says the orch takes the step from the record at parent review.
- goal-scope: the verb is unasked — fixed: the Goal says why a tracked hub-wide file needs it.
- goal-scope: `CLAUDE.md` wording dropped the own file — fixed: D5 names both files.
- goal-scope: the edited-board-copy risk does not hold today — fixed: the risk says today's copy is unedited.
- goal-scope: the Production rule may be stale — fixed: an open risk leaves it to the operator.
- goal-scope: the first criterion was too broad — fixed: tracked sources, stencils and docs outside Scope's exclusions, with generated content excluded.
- leanness: Auto-mode assumptions repeat the decisions — fixed: one pointer line.
- leanness: Constraints repeat D1's rationale — fixed: D1's rationale drops the pattern names.
- leanness: D3 rationale clause belongs in rejected alternatives — fixed: replaced by the restated rationale.
- leanness: the plan-writer note repeating D4 — fixed: cut.
- leanness: the criterion restating Scope's exclusions — fixed: it points at Scope.
- pattern-fit: fabric-git's positive-only file list unstated — fixed: Constraints say it, as `configcli`'s hub writes already do; no PATTERN change.
- pattern-fit: stencil refresh happens at the first stamped process, not at deploy — fixed: Constraints wording.
- pattern-fit: no test tier for the verb — fixed: plan-writer note.

## Rejected alternatives

- Edit the hub-wide file on Bolt in place, then a `commit` verb: leaves Bolt dirty between the edit and the commit, which can block the board's own pull, and leaves an uncommitted file when the commit is forgotten.
- No verb, the orch commits Bolt with raw git: races the board write lock and bypasses `fabricengine`.
- Keep one `repository_rules` marker filled with both files joined: hides which file a rule came from and needs a joining convention in Go.
- Put the rules in the role stencil's board copy: `lyx stencil` reads that edit as drift to port back to the shipped source (board entry's orch recommendation).
- A card writes the moved rules: a card can reach Bolt only through `lyx board`, and the verb that commits the rules file is not deployed until the task lands.
- Strip `_lyx/orch/rules.md` from a new pair's weft: no reader exists outside the prime today, so it solves nothing this task needs; recorded as an open risk for `orch-any-worktree`.

## Review rounds

### Round 1

- F1: no landing path gives `ly:orch` a turn with the record, and `alignment_pr_approve` is a guard key `run_settings` cannot set — fixed: D4 names the operator as the actor, and the open risk says the text survives in the record and git history.
- F2: `lyx orch stop` removes the orch's own strand, and the Linux watcher re-execs onto the deployed binary — fixed: the restart step is dropped, and the open risk bounds the Linux window and gives the Windows restart to the operator.
- F3: `configcli` restores only inside the write closure — fixed: D3 and its criterion report a failed commit as uncommitted.
- F4: `isRequestedNote` uses `RepoRulesPath`'s directory as the prime root — fixed: D1 adds a told `PrimeRoot`.
- F5: a draft holding only a new rule deletes the others — fixed: D3's help text and D5's placement sentence say the verb replaces the whole file.
- F6: a commit over a stale Bolt copy conflicts at every later push — fixed: D3 writes through `PullThenCommitWritten`, with a criterion for the skip.
- F7: unchanged content after a failed push — fixed: the verb pushes before its pull-first write and after it (Q8), with a criterion.
- F8: the prime-only refusal is a working-directory check — fixed: D3's bound states it and names the preamble as what bounds the text.
- F9: the temporary file is a second Bolt path — fixed: D3 states the exposure and the bound commits only the rules path.
- F10: the `###` heading rule has no carrier — fixed: D2's moved preamble states it.

### Round 2

- F1: `lyx board` writes Bolt from a task pair — fixed: D4 rests on the undeployed, prime-only verb, a card files the `orch-rules-move` board note, and the no-reminder risk is gone.
- F2: the failed-commit recovery was the next board sync's sweep — fixed: D3 and its criterion name re-running the verb.
- F3: the restore on a failed write guards nothing after an atomic rename — fixed: cut from D3 and the plan-writer notes.
- F4: D2 stated the heading rule twice — fixed: the second statement is cut.

## Question ledger

- Q1 Where today's two rules go — auto-pick: hub-wide file (D4).
- Q2 How the hub-wide file is committed — auto-pick: `lyx orch rules --from <file>` (D3).
- Q3 How the own file is committed — auto-pick: existing `lyx fabric commit`/`sync` (D3).
- Q4 Render shape — auto-pick: two closing H2 sections, hub-wide first (D2).
- Q5 Who moves the content — auto-pick: `ly:orch` after landing and deploy; changed in round 1 to the operator, since no landing path gives the orch a turn (D4).
- Q6 Empty `--from` content — auto-pick: accepted, as the way to clear the hub-wide rules (D3).
- Q7 A failed push after a landed commit — auto-pick: keep the commit and report it as not pushed (D3).
- Q8 Whether unchanged content pushes — auto-pick: the verb pushes before its pull-first write and again after it, so every successful run, a retry included, carries an unpushed commit (D3).
