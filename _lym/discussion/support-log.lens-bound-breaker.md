```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker review of the raddle decision record and support log.

**Findings**

1. **BLOCKING. D9 and D14: the stored records do not survive a prune.**
   - D9 says the `tasks` shape marks the Sketch entry done "keeping its record", and the support log says this keeps it linked from the README.
   - D14 reads each stored proposal after its parts land.
   - Counterexample: the raddle run's own batten Land row calls `board.Prune(true)` (`internal/battencli/land.go`, the land board prune). That removes every done entry.
   - Removing an entry deletes its `decision-<slug>.md` (`internal/boardengine/decision.go`, the loop over `loaded` entries not `kept`).
   - So in the `tasks` shape the Sketch entry and its record are gone at the raddle's own landing.
   - In every shape, the filed parts' entries and part records go at their own landings' prunes. So does the tapestry-shape entry, and the project at the beam's landing.
   - A `raddle-<slug>.yaml` tied to the entry's lifecycle is either deleted the same way, which breaks D14's "after landing" comparison, or orphaned, which the record never says.
   - The record needs a store for proposals and their part slugs that outlives prune, or D14 must read them from Bolt history.

2. **MEDIUM. D8: a new label-sharing entry can never be cleared.**
   - The rule "no open entry created after the cold proposal carries a module label the Sketch carries" is measured from the cold proposal.
   - Counterexample: in an interactive session, another run files a note labelled `loom` after the cold proposal. The agent reads it, adds it to `board_checked` and revises. `raddle-approve` still fails, because the entry was still created after the cold proposal. No revision can ever pass.
   - The rule also counts notes. Runs and orchs file notes with broad module labels such as `loom` or `board` all the time, so a long interactive session or an auto run gets returned constantly and hits the 2-return halt.
   - Fix: measure against the latest proposal's gate pass, exempt entries already listed in `board_checked`, and consider only tasks and projects.

3. **MEDIUM. D7: the bound leaves out that the agent can approve itself.**
   - The bound says `lym loom raddle-approve` refuses only outside Raddle-Write, on an invalid proposal or on a moved board.
   - Counterexample: the raddle agent itself, or any session at that row, runs the verb with no operator input. The approval is recorded and Raddle-File files.
   - "Nothing is filed before approval" holds only by the stencil's instruction. The bound must say the verb lets the agent's own call through, as `lym loom sampler-done` does, or tie the approval to the operator.

4. **MEDIUM. D2: one promotion cannot set the recipe.**
   - `Board.Promote` takes only an id or slug and sets no recipe (`internal/boardengine/board.go`).
   - The board rule "admits `sketch` on a task only while its recipe is `raddle`" then refuses a plain `lym board promote` of a Sketch. Or, if the recipe is set in a second write, dobby could start the entry in between with the board's default recipe.
   - D2 must name the one write that sets kind task and recipe `raddle` together, such as an upsert, or extend promote to carry a recipe. It must also say what status the entry gets, draft or released.

5. **LOW. D8: pruned entries fail the presence check.**
   - "Every `board_checked` entry is present" breaks when a weighed peer lands and its Land prunes it during the session.
   - That counts as a moved board, although the peer landing satisfies any dependency on it.
   - The check should treat a weighed entry that is now done or pruned as unchanged, or name it to the agent as landed.

**Claims attacked**

- D9's bound "Raddle-File writes as Go code with no strand name, so the agent's `lym board` calls stay refused" held. `checkTaskCreation` keys on `cfg.StrandName`, and `OpenHub` carries none.
- D8's `revised` stamp held against run-status churn. Dates bump `Revised` only on `scopeChanged`, which skips Status, so a peer's row moves do not count as a moved board.
- D7's hash binding of edits after approval held.
- D9's idempotent retry held.
- D11's premise held: `NotifyOrchAt` posts nothing to a worktree with no orch strand.
- D9 "the run lock admits this one change" held in principle, by the `endsAsSketch` precedent.
- D10's clearing generalized by rows is plausible. It is untested whether clearing runs on a plain start as well as on `lym beam create`; the record says both.
- D9 "marked done, keeping its record" broke (finding 1).
- D14 "after landing" broke (finding 1).
- D8 "created after the cold proposal" broke (finding 2).
- D8 "every `board_checked` entry present" broke (finding 5).
- D7's approval bound broke (finding 3).
- D2's promotion path broke (finding 4).

**Tool calls**

- Read `internal/boardengine/taskguard.go` and grepped strand-name use: needed to see how the board detects a run agent, which the context only summarized.
- Grepped `Revised` in boardengine: needed to know what bumps the stamp D8 relies on.
- Read `internal/battencli/land.go` (lines 120–175): needed to see what a no-code run's Land does to the board.
- Grepped boardengine for decision-file removals: needed to find whether removing an entry deletes its record.
- Read `internal/boardengine/decision.go` (lines 104–122): needed to confirm the record is deleted when its entry is removed.
- Read `Board.Promote` and `Prune` in `internal/boardengine/board.go`: needed to see whether promote can carry a recipe.
