# Add the raddle recipe: turn a Sketch into tasks, a project or a beam

A new record-only loom recipe, `raddle`, runs on a discussed Sketch and files what the operator approves: one tapestry, several tasks ordered by `depends_on`, a project, or a beam.
A cold Opus agent proposes the split from the Sketch, the board, a bounded read of code and the hub's own measured run costs.
The operator approves the proposal in the agent's pane, or `raddle_auto_approve` skips the approval, and only then are entries filed on the board.
Recording every agent session's cost is part of the change, so the proposal's cost figures come from the hub's own runs and each proposal can later be compared with what its tasks cost.

No issues are closed.

## What changes

- **Recipe.**
  `contracts/recipes/raddle-recipe.yaml` has the rows Record-Write, Raddle-Write and Raddle-File and no build, review or Finalize row.
  Batten runs it like `motif`: its own worktree, a pane, a board status and notices, no code landed.
  `RaddleWrite` (agent) and `RaddleFile` (Go) are `Constructor` rows in `internal/shedrecipe`.
- **Start.**
  `lym board promote` takes an optional `--recipe`.
  A write making a discussed note with a stored record a task must set recipe `raddle`, keep the record and the `discussed` flag, and carry the `sketch` label; any other such write is refused, naming `lym board promote <slug> --recipe raddle`.
  `boardengine.IsSketch` reads a `raddle` task as no Sketch.
  Batten's start guard refuses a raddle run on an entry with no stored record (naming `motif`), and on a slug the raddle store holds for another Sketch or as a filed slug (naming `lym board rename`).
- **Record-Write** copies a discussed entry's stored record into the run's decision record instead of rendering it from the body.
  This holds for every recipe with that row, so a darn entry flagged `discussed` now builds from its stored record.
- **Agent and proposal.**
  Stencil `contracts/stencils/raddle/raddle-template-agent.md`, loom key `raddle` (default `opus[high]`).
  The proposal is `_lym/raddle/proposal.yaml`, specified in `contracts/specs/raddle-proposal-spec.md` and parsed only by the new `internal/raddleparser`.
  The stencil carries the rule that a tapestry's fixed start cost often favours fewer, larger parts, and no cost figure.
- **Approval.**
  `lym loom raddle-propose` runs the gate checks, stores the cold proposal on the run's first pass and posts the waiting notice on the first pass of each Raddle-Write session.
  `lym loom raddle-approve` re-runs the checks and the board re-check and records the approval with the proposal's hash.
  `raddle-approve` refuses until the current session has a passing `raddle-propose`.
  Auto mode runs the same gate on Raddle-Write's handoff, re-prompting up to 3 times and then halting.
  The mode is read once at Raddle-Write's first spawn and kept in the run's state.
- **Filing.**
  Raddle-File re-checks the board, stages the store and per-part records (spliced by a new `internal/discussionparser` function), files all entries in one `boardengine` write with each record stored and `discussed` set, creates the beam for shape `beam`, releases its parts, sends the notice, and marks the run's entry done last.
  Every step is skipped when its result is already there, so a retry after a crash finishes only what is missing.
  A moved board, an invalid proposal or an approval-hash mismatch returns to Raddle-Write; the third return halts the run.
- **Raddle store.**
  One directory per Sketch beside `board.json`, outliving prune, holding the approved and cold proposals, the Sketch's record and `created` stamp, and the filed slugs.
  `checkNewSlugs` also refuses a slug the store holds, `FabricReplay` replays store files, and `lym board raddle <slug>` prints a stored proposal.
- **Beam hand-off.**
  Beam-Up mails the beam's orch the Sketch slug and the proposal, then records the mail as sent.
- **Usage and reports.**
  `shuttleengine.SessionUsage` gains the four token counts, and the Codex engine gains a `UsageReader`.
  One writer in shed records each session as `_lym/usage/<session>.json`, priced per message through the engine's `MessagePricer`.
  `lym report cost` gives per recipe, profile and stored-record flag the run count, the mean and a fixed-plus-per-100-lines fit; `lym report raddle` compares each proposal's predicted cost with its landed parts.
- **Docs.**
  The glossary marks `raddle` as taken.
  The `PATTERN.md` overview lines are trimmed for headroom, with the full rules moved to the `pattern/` background files.

## Design decisions to know

- Filed entries take the Sketch's own `project` field, not `hubgeom.ParentOf`, which skips a grouping project.
- A filed project's type label comes from a required `type` key in the proposal's `project` object, since the Sketch's own `sketch` label is not allowed on it.
- The run's entry stays open until Raddle-File's last write, because origin reads a done entry as an ended run whose resume is refused.
- The board keeps no status for a removed entry, so a `depends_on` on an entry gone from the board is treated as landed and dropped at filing.
- The report config lives in `internal/reportconfig` to avoid an import cycle with `configreg`.
- The verbs cannot tell the operator from the agent, so an approval with no operator input is prevented only by the stencil; the auto key is the sanctioned way to file without one.

## To check by hand

- `internal/loomshed/raddlefile.go`, `internal/boardengine/raddlefile.go`: a half-done filing is resumed on retry for the staged proposal only, and the done marker is covered by a test.
  An earlier review found both missing; later commits address them, so confirm.
- `docs/overview.md`: the raddle run directory and the `_lym/usage/` directory are described and the usage row is in the overview.
  An earlier review found these stale.
- A hub has no usage records until runs land under this binary, so early `lym report cost` cells read "too few runs".
- A Codex session's dollar cost stays unknown because Codex has no `MessagePricer`; its runs enter `lym report cost` in hours alone.
- The plan-level verify run failed once on `internal/configsync.TestReconcileHubWideAt_SeedsFromHubPrimeLegacyOrTemplate` and `internal/boardcli.TestCLI/Profiles`.
  Commit `8889afb0` fixed them by expecting the raddle profile default; confirm the assertions still mean what they did.
- Not run end to end against a live hub: an interactive raddle run through approval and filing, and a `beam` shape run through Beam-Up's hand-off.
