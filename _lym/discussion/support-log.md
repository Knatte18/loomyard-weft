# Support log — pawl-checklist-hidden-in-beam

## Interview

Autonomous run: no operator answered; every pick below is an auto-pick, recorded in the question ledger.

### Turn 1 — scope from the board

The entry names the cause (`ensureRunsStrand` replaces the loom status strand, since reed refuses a second overview strand) and the orch's recommended fix: let the overview anchor hold a second strand in a beam's session, beam overview first, checklist second.
The title commits to that design.
No `Decision for pawl-checklist-hidden-in-beam:` note is on the board, and no other entry touches reed's overview anchor, the runs strand or the loom status strand; E8 and E11 carry the pawl label but concern beam creation and parent merges.

### Turn 2 — code read

- reed refuses a second overview strand in `AddStrand` and `ReplaceStrand`; overview windows open before the lowest-index window, so a later-added overview would always go first.
- `relevantWindow`, `followTargetWindow` and the attach selection each fall back to the one overview window.
- `statusStrandAddSpec` switches to an own window with `KeepViewer` when a runs strand is present; `ensureRunsStrand` replaces a loom status strand when no runs strand is present.
- The pawl run's first `lym loom start` comes from `lym beam create`'s detached pass, before `lym beam start` adds the runs strand, so ordering the starts cannot fix it in every case.
- `lym batten overview` names nothing of the beam's own pawl run.

### Turn 3 — design picks

Picked an order key on the overview display, zero value first, with ties in opening order and no cap (D1); the lowest-index overview as the focus fallback (D2); the status strand always an overview, key after the runs strand's, titled with the recipe name (D3); the runs strand added beside it (D4); no migration (D5).

### Turn 4 — lens review

Six forks returned reports; none was missing.
Facts checked before judging: `lym reed add` refuses the overview anchor (`internal/reedcli/add.go`); no shuttle caller sets it; `seedAndCommitBootstrap` writes the seed through `resolveSeedRecipe`, the read `resolveRecipe` makes; the batten pass's loom session spawn runs `lym loom start --no-attach`.

- bound-breaker 1, leanness 1, walkthrough F3 (recipe title on a fresh start, deferred to Plan): fixed; D3 states the evidence that the held recipe equals the seed's on every start, and the notes bullet is cut.
- existing-paths 1, bound-breaker 2 (D1 bound names the wrong guard): fixed; the bound names the CLI's engine-only refusal and that no shuttle caller sets the anchor. existing-paths' claim that `lym reed add --anchor overview` reaches reed is rejected: add.go refuses it.
- interactions 1, existing-paths 2, bound-breaker 3 (an update can change the order key without moving the window): fixed; `UpdateStrand` refuses an order-key change, with an acceptance criterion.
- interactions 2 (tied keys reorder on replace and relaunch): fixed; D1 states ties sit in the order they last opened, the replace criterion is scoped to distinct keys, a tie criterion added, and D5's bound names replace and relaunch.
- walkthrough F1 (who runs the healing loom start): fixed; D5 names the batten pass and a hand `lym loom start` in the beam's anchor, which is re-entrant.
- walkthrough F2, existing-paths 4 (D2 wording per source, `agentWindowPanes`): fixed; D2 words reed state for `relevantWindow` and the marker for follow.
- existing-paths 3 (an older live lym process, dropped key): fixed; D5's bound names both.
- leanness 2, 3 (padding constraint, repeated genericity): fixed; the pane-text line is cut and the genericity clause kept only as a constraint.
- leanness 4 (Scope In restates the decisions): fixed; Scope In points at D1 to D5.
- leanness 5 (pawl row line stated twice): fixed; kept in Scope Out only.
- leanness 6 (Open risks duplicates an auto-pick): fixed; the line is dropped, the auto-pick stays.
- leanness 7 (notes bullet repeats relevant.md): fixed; shortened to the doc paragraph names.
- leanness 8 (D1's "everything else stays" belongs in Constraints): fixed; moved.
- board-scope: no findings.

## Rejected alternatives

- Re-add the checklist in an own window after the runs strand replaces it: it keeps a coloured agent-style window that the focus rules treat as an agent window, and the operator asked for a second overview window.
- Bring the runs strand up before the pawl run's loom start: `lym beam create` starts the pawl run's first pass before any `lym beam start`, so the order cannot be held.
- A boolean "lead" flag in place of an order key: it fixes two places only; an integer key costs the same and needs no second change for a third overview.
- Keep a refusal on equal order keys: an overview strand recorded by an older lym reads as key zero and would block the runs strand's add in an existing beam session.
- A one-off migration of existing sessions: the status strand's keep, replace or add at every loom start already moves it.
- Branch the window title on whether the worktree is a beam: the recipe name is the run's own record and needs no branch.
- Add a pawl row line to the beam overview: optional in the entry and duplicated by the checklist window.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): one overview window split in two panes, or two overview windows? Two windows, per the entry's recommendation; panes get narrow on a phone.
- Q2 (resolved, auto-pick): how does reed order several overview windows? An integer order key on the display, zero value first, ties in opening order (D1).
- Q3 (resolved, auto-pick): does reed still cap overview strands? No; each spawner's explicit role prevents duplicates (D1 bound).
- Q4 (resolved, auto-pick): which overview does a focus fallback pick? The lowest-index one (D2).
- Q5 (resolved, auto-pick): what is the checklist window named? The run's recipe name, in every session; the task-session rename recorded as an auto-mode assumption (D3).
- Q6 (resolved, auto-pick): the pawl row line in the beam overview? Left out (Scope).
- Q7 (resolved, auto-pick): migrate existing beam sessions? No; the next loom start heals them (D5).
