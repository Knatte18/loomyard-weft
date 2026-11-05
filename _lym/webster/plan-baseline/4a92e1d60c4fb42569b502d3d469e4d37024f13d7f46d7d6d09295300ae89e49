# Card 20 — raddle-verbs

**Create:**
- `plan:internal/loomcli#raddleVerbDeps` -> `type raddleVerbDeps struct`
- `plan:internal/loomcli#raddleProposeVerb` -> `func raddleProposeVerb(out io.Writer, deps raddleVerbDeps) int`
- `plan:internal/loomcli#raddleApproveVerb` -> `func raddleApproveVerb(out io.Writer, deps raddleVerbDeps) int`
- `plan:internal/loomcli#loomCLI.raddleProposeCmd` -> `func (c *loomCLI) raddleProposeCmd() *cobra.Command`
- `plan:internal/loomcli#loomCLI.raddleApproveCmd` -> `func (c *loomCLI) raddleApproveCmd() *cobra.Command`
- `plan:internal/loomcli#raddleBoardContext` -> `func raddleBoardContext(location *lymcwd.Location, filing bool) (raddleparser.BoardContext, error)`
- `plan:internal/loomcli#raddleCheckFor` -> `func raddleCheckFor(location *lymcwd.Location) loomshed.RaddleCheck`
- `plan:internal/loomcli#TestRaddleProposeVerb` -> `func TestRaddleProposeVerb(t *testing.T)`
- `plan:internal/loomcli#TestRaddleApproveVerb` -> `func TestRaddleApproveVerb(t *testing.T)`
- `plan:internal/loomcli#TestRaddleBoardContext` -> `func TestRaddleBoardContext(t *testing.T)`

**Edit:**
- `internal/loomcli#Command`
- `internal/loomcli#verbUsesLightweightWiring`

**Uses:**
- `internal/loomcli#samplerDoneVerb`
- `internal/loomrecipe#EngineRow`
- `plan:internal/loomshed#CheckRaddleProposal`
- `plan:internal/loomshed#RecheckRaddleBoard`
- `plan:internal/loomshed#UpdateRaddleState`
- `plan:internal/loomshed#StoreColdProposal`
- `plan:internal/raddleparser#Hash`
- `internal/orchcli#NotifyParent`
- `internal/boardengine#ReadTasks`
- `internal/runprofile#Load`
- `plan:internal/loomengine#RaddleProposal`
- `plan:internal/loomengine#RaddleColdProposal`
- `plan:internal/loomengine#RaddleApproval`
- `plan:internal/loomengine#RaddleStatePath`

**Intent:** D7's interactive verbs, `lym loom raddle-propose` and `lym loom raddle-approve`, modelled on `sampler-done`, and the board context every raddle check reads.
`raddleBoardContext` builds `raddleparser.BoardContext` for the run at location: the Sketch is the run's own entry (`seedSlug`), its module labels its non-type labels, every board entry from `boardengine.ReadTasks` of the hub's board, the store's held slugs from `ReadRaddleStore`, the board's type labels from its config, the profiles from `runprofile.Load`, and `filing`.
`raddleCheckFor` assembles the `loomshed.RaddleCheck` of a run from card 9's paths and `raddleBoardContext`.
Both verbs refuse, before reading anything else, a run whose recipe has no `RaddleWrite` row (`loomrecipe.EngineRow`, so the refusal is by the recipe's rows, never its name, `PATTERN-loom-contract`) and a run whose current row is not that row, each naming the way forward as `sampler-done`'s do; their reaching input is either verb run in a darn run's worktree, or in a raddle run standing at Raddle-File.
`raddle-propose` runs `CheckRaddleProposal(check, false)` and prints the findings; on a pass, the run's first passing call stores the proposal as `cold-proposal.yaml` and sets `cold_stored`, which no later call overwrites; every pass records `proposed_session`, and the first pass of a Raddle-Write session (`notice_session` behind `session`) posts the parent orch, through `orchcli.NotifyParent`, a notice that a proposal for the Sketch waits for the operator, then records `notice_session`.
`raddle-approve` refuses, each naming what to fix: a current session with no passing propose (`proposed_session` behind `session`, the first session after a return included), naming `lym loom raddle-propose`; an invalid proposal, with its findings; a moved board, naming the moved entries from `RecheckRaddleBoard`; on success it writes `approval.json` with the proposal's `Hash` and the session, records `approval`, and prints the landed entries the re-check named.
Bound: the verbs cannot tell the operator from the agent, so, as with `sampler-done`, an approval with no operator input is prevented only by the stencil; `raddle_auto_approve` is the sanctioned way to file without one.
`Command` registers both; `verbUsesLightweightWiring` keeps them on the wiring `sampler-done` uses.
`TestRaddleProposeVerb` (tier0, injected deps) has rows: no RaddleWrite row (refused); current row elsewhere (refused); findings (printed, nothing stored); first pass (cold stored, notice posted); second pass same session (cold kept, no notice); first pass of a later session (notice posted again, cold kept).
`TestRaddleApproveVerb` (tier0) has rows: outside Raddle-Write (refused); no passing propose this session (refused naming propose); a passing propose in session 1 only, the state now at session 2 after a return (refused naming propose); invalid proposal (refused with findings); moved board (refused naming entries); success (approval written with the hash, landed entries printed).
`TestRaddleBoardContext` (tier1, a `hubforge` hub) builds the context of a raddle entry and asserts its module labels, entries and store slugs.

**ImpactSummary:** Two new role verbs and one context builder; the verb registration list and the lightweight-wiring list each gain two names.
