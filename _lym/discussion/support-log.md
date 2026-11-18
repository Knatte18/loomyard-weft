## Interview

Autonomous run: no operator turns. Each design point was settled by an auto-pick, listed in the question ledger.

### Turn 1 — exploration

- Read the board entry and the board list; no `Decision for wxb-plan-with:` note exists. The entry is a bench arm of `plan-writer-explorer-forks`, which owns the bench itself; no other entry touches beam session windows.
- Read `internal/orchcli/runs.go`, `internal/loomcli/bootstrap.go`, `internal/loomcli/sharedbootstrap.go`, `internal/loomcli/statusbuild.go`, commit `fa38b1ef`, the pawl wiring, the batten overview and reed's own-window placement.
- Facts settled from the code: `reedengine.StrandStatus` carries no command or display, so orchcli cannot copy a live strand's spec; loomcli imports orchcli, so the shared builder goes in loomengine; reed names an own window by role and reads `Display.Title` only for the overview; the pawl run's Beam-Up row calls `orchcli.BeamUp` from inside the live run.

### Turn 2 — lens review

Every lens returned a report.

- walkthrough 1 and bound-breaker 1 (fallback targets a GUID a failed replace already removed): fixed, D2 reads the strands again after a failed move and replaces or adds.
- bound-breaker 2 and interactions 2 (concurrent `lym loom start` race): fixed, D2 names the outcome and `lym beam start`.
- interactions 3 (no rule for a failed runs add after the move): fixed, D2 failure bullet and criterion.
- walkthrough 2, bound-breaker 3 and existing-paths 1 (reed restart leaves the checklist dead): fixed, D2 case 2 relaunches a dead `loom-status` beside a runs strand.
- walkthrough 3 and bound-breaker 4 (move repeats on an own-window `loom-status`; "in the overview window" not checkable): fixed, D2 case 1 states it and the criterion drops the precondition.
- bound-breaker 5 and existing-paths 3 (viewer landing unstated): fixed, D2 states the viewer ends on the beam overview, with a criterion.
- bound-breaker 6 and existing-paths 4 (title colliding with a reserved window name): fixed, D3 falls back to the role name.
- existing-paths 2 and interactions 4 (loomcli's recipe defaults to tapestry; builder's recipe input implicit): fixed, D1 reads the seed in both callers with no default.
- interactions 1 (overview line may point at a missing window): rejected, the missing-window cases are warned failures with a way forward, and checking reed from every frame costs more than the rare case.
- board-scope 1 (parent commit `7c23591` fixes the bug another way): raised under Open risks as a question for the parent.
- board-scope 2 (record does not say it is a bench arm): fixed, folded into the same Open risk.
- leanness Open risks (older-build line is no question): fixed, moved to Scope Out.
- leanness Scope In restating D1 to D4: fixed, cut to one line.
- leanness Auto-mode third bullet: fixed, cut.
- leanness Goal cause sentence: rejected for Goal, kept as the problem statement; the duplicate in D2's rationale was cut instead.
- leanness D2 rationale sidecar sentence: fixed, cut.
- leanness Constraints `PATTERN-loom-contract` and named-constants bullets: fixed, cut.
- leanness Notes for the plan writer: fixed, section cut and `overviewWindowTitle` moved to `relevant.md`.
- leanness D4 with no seed: no change, D4 already draws no line without a seed.

## Rejected alternatives

- Bring the runs strand up before the pawl run's `lym loom start`: the beam is marked and brought up by the pawl run's own Beam-Up row, after loom start; reordering would make loom start own orch's runs strand.
- Put the pawl checklist in its own window from the first `lym loom start` by recipe: keys placement on a recipe, against `PATTERN-loom-contract` and `PATTERN-no-hard-coded-choice`, and leaves older sessions broken.
- Leave the move to the next `lym loom start` or resume: that is today's behaviour and the bug.
- Skip the sidecar write after the move: the next loom start would relaunch the checklist pane for no reason.
- Name the window by a new role: the role `loom-status` is pinned for lookups.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Where does the fix go? Auto-pick: in `ensureRunsStrand`'s replace path, moving the checklist out (D2). Resolved.
- Q2 How does orchcli build the checklist strand? Auto-pick: shared spec in loomengine (D1). Resolved.
- Q3 What is the window named? Auto-pick: the run's recipe name from its seed, through `Display.Title` extended to own windows (D3). Resolved.
- Q4 Is the overview line in scope? Auto-pick: yes, one line for the worktree's own live run (D4). Resolved.
- Q5 What when the move fails? Auto-pick: warn, name `lym loom start`, fall back to today's replace so the overview still comes up (D2). Resolved.
