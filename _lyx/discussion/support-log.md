# Support log — hub-workspaces

## Interview

### Turn 1 — context gathered, no operator

The session ran autonomously with no advisors.
The board entry already held the operator's decisions on location, naming, folders, display names and order; the open points were where regeneration lives, when it runs, local pairs, concurrency and what happens to `_portals` on existing hubs.
Read: `internal/fabricengine/hubworkspace.go`, `hubworkspace_write.go`, `portals.go`, the portal call sites in `add.go`, `remove.go`, `prune.go` and `reconcile.go`, `internal/ideengine/spawn.go`, `internal/vscode/workspace.go`, `lyxcwd.Location`, `PrimeName`, `CodeWorktrees`, the mutation kinds, and the related board entries `orch-any-worktree`, `pair-from-parent`, `umbrella-anchor` and `fabric-remote-async`.
Every open point was settled by auto-pick (see the question ledger).

### Turn 2 — lens review

All six forks returned.

- goal-scope MEDIUM, no file on a fresh hub: fixed, `Clone` regenerates (D4) and a criterion covers it.
- goal-scope MEDIUM, migration step nobody is told about: fixed, regeneration warns while retired artifacts exist (D7).
- goal-scope LOW and premises LOW, Reconcile folded under the wrong rule and Cleanup left to the plan: fixed, Reconcile named as the repair verb, Cleanup removes no worktree and does not regenerate (D4).
- goal-scope LOW, umbrella file should reuse D3: fixed in Scope Out.
- goal-scope LOW and board-fit MEDIUM, prime file lists pairs forked from pairs and the umbrella handoff is undocumented: fixed, D2 states it and `fabricengine`'s doc records that no partition exists; filing a note on `orch-any-worktree` rejected, since its own "Create: its pair, thread and workspace" step already owns per-umbrella files.
- board-fit MEDIUM, batch worktrees would flood the workspace: rejected as a design change, no batch worktree exists and `parallel-batch-parts`/`weaver` decide their listing; named in Scope Out.
- board-fit LOW, umbrella is now beam: fixed, Scope Out names both and `umbrella-daemon`.
- board-fit LOW, `--local` may be removed later: rejected, listing it costs that removal one test row.
- leanness MEDIUM, D6 symbol list: fixed, cut to the surfaces; `relevant.md` keeps the names.
- leanness LOW, D8 doc list, D2 editing note, generic constraints, auto-mode restatement, open-risk duplicate, Notes overlap: fixed.
- scenarios MEDIUM, prime resolution failure: fixed, D3 writes nothing and returns the failure.
- scenarios MEDIUM, unbounded lock wait: fixed, bounded leaf lock (D5) and a criterion.
- scenarios MEDIUM and pattern-fit MEDIUM/LOW, temp file left by a crash, `PATTERN-lyx-temp-root` departure, temp cleanup outside `destroy.go`: fixed by dropping the atomic replace (D5).
- scenarios MEDIUM, sweep from a non-prime worktree and unresolvable prime: fixed in D7.
- scenarios MEDIUM, dangling portal links: fixed, matched by target path whether or not it exists (D7), with a criterion.
- scenarios MEDIUM, refused or partial Remove: fixed, regeneration on every exit past pre-flight (D4).
- scenarios LOW, empty hub and missing `_board`, unreadable settings, planted link at the retired file: fixed with criteria and D3.
- scenarios LOW, empty Interview turn: fixed by this turn.
- premises MEDIUM, `CodeWorktrees` fails on one unresolvable entry: fixed, such an entry is skipped (D2).
- premises LOW, hub-dir scans note: fixed, premise stated in D1 and the note dropped.
- premises LOW and pattern-fit MEDIUM, ide spawn has no recorder: fixed, it passes a discarded one (D3).
- pattern-fit MEDIUM, hub-containment change mixes rules and background left vague: fixed, entry text reworded and background wording given in Notes.
- pattern-fit LOW, `_portals` constant owner: fixed in D6.
- pattern-fit LOW, `vscode` import drags launch code: rejected, `internal/vscode` is a small leaf-ish package and no rule pins `fabricengine`'s imports.
- pattern-fit LOW, lock ordering: fixed, leaf lock (D5).

## Rejected alternatives

- Keep the regeneration in `ideengine` and call it from `fabriccli` and `battencli` after each verb: misses every in-process caller of the fabric verbs and spreads one rule over several call sites.
- Keep `fabricengine` VS Code-agnostic by writing the JSON itself: duplicates the format knowledge `internal/vscode` owns.
- Keep `_portals` alongside the new file: the operator decided it is not needed, and it would keep a second hub-level surface alive with no consumer.
- Leave the old portals and workspace file for the operator to delete by hand: dangling links pile up as pairs are removed, and nothing tells the operator.
- Sweep retired artifacts on every regeneration: puts a destructive step into every `add`; once per hub through reconcile is enough.
- Leave local pairs out: that rule served batch pairs, which `fabric-remote-async` withdrew.
- Write the file per umbrella now: no umbrella besides the prime exists; the umbrella tasks own that.
- Atomic replace through a temp file in the hub: needs a same-dir temp file that departs from `PATTERN-lyx-temp-root` and a cleanup outside `destroy.go`, for a partial-read window VS Code recovers from anyway.
- Exclude short-lived batch worktrees now: none exist, and the rule belongs to the task that creates them.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Where does regeneration live? Auto-pick: one `fabricengine` function importing `internal/vscode`, called by the verbs and the prime spawn (D3).
- Q2 Which verbs regenerate? Auto-pick: every verb that creates or removes a warp task worktree, best-effort (D4).
- Q3 Are local pairs listed? Auto-pick: yes (D2).
- Q4 How are concurrent writers handled? Auto-pick: bounded leaf hub lock around list and write, in-place write (D5); the atomic replace was dropped at lens review.
- Q9 Does a fresh clone write the file? Auto-pick: yes (D4).
- Q10 What does a failed prime resolution do? Auto-pick: write nothing and return the failure (D3).
- Q5 What happens to existing portals and the old file? Auto-pick: `lyx fabric reconcile` sweeps fabric-owned ones (D7).
- Q6 Does `_portals` leave the reserved name set? Auto-pick: no, existing hubs still hold it (D6).
- Q7 Prime display name on detached HEAD? Auto-pick: its directory name (D2).
- Q8 Which pairs belong to the prime's workspace? Auto-pick: every task pair of the hub, since the prime is the only umbrella (D2).
