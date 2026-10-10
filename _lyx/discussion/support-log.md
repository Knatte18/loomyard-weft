# Support log — MultiLLMProducer

## Interview

Autonomous mode: no operator answered; every turn below is the agent's own exploration and pick, recorded as such in the question ledger.

1. Read the board entry and the entries that touch it: `burler-multi-llm`, `discussion-multi-llm`, `plan-multi-llm`, `creel`, `reed-layout-shares`, `shuttle-state`, `shuttle-send`, `run-message-routing`, `extra-agent-strand`, `review-loop-convergence`, `reed-view`.
   Took the operator's fixed terms (seat, chair, advisor-n, strand names, no "master") and the fixed shape (inside one producer, standalone, no consumer, own channel, no creel) as given.
2. Explored the module boundaries: `shedadapters` (adapters over engines; `SingleLLMProducer`'s probe-before-archive and outcome mapping), `shedengine` (producer contract), `shedrecipe` (registry, `Env`, config accessors, `SingleLLM` row), `shuttleengine` (`Spec`, `Run` handle with `Wait`/`Stop`/`Send`, held-turn notice, session-state vocabulary), `burlerengine` (two-half round: start, join, stop, probe, resume), `stencil` (`Fill`, no include), `stencilstore` (name to path by first dash segment, no registration needed to read), `agentname` (role grammar, role constants rule), `reedengine` (one layout rule, name formed once, `-N` numbering), `parentdirective` and `editdirective`, `contracts/stencils` (registry, opening-stencil guards), `modelspec`, test kits and the test budget.
3. Settled the layering: runner below the adapter, because `shedadapters` is a translation layer and `burlerengine` must be able to import the runner later.
4. Settled the table: one Go type, one recipe form, construction-time rules including advisor-outputs-are-chair-inputs.
5. Settled stencils: one per seat, shared blocks by `{{template}}`, one level, marker guarantee extended; two shipped blocks carry the channel words.
6. Settled the channel: Claude Code's session message by strand name, chair-centred, advisors never peer-to-peer; `creel` later.
7. Settled failure rules: the chair decides; an advisor's loss is told to the chair and never ends the step; hangs are the chair's call.
8. Settled resume: probe every seat before archiving; a dead chair ends the dialogue; advisors first, chair last, which also gives the chair the expanded pane under reed's current rule.
9. Settled `SingleLLMProducer`: unchanged; N = 1 pinned by a test.
10. Settled held-turn notices: advisors quiet, chair as today.
11. Listed the test tiers, the budget rows and the docs to land in the same commit.

## Rejected alternatives

- **Everything inside `internal/shedadapters`.** The board names the package for the producer, and Bouncer is already an exception to "thin adapter".
  Rejected because `burler-multi-llm` needs the runner below `burlerengine`, which cannot import `shedadapters` without a cycle, and because a start/join/probe/resume loop is engine logic by this repo's own split.
- **Shared blocks as fixed markers filled with stencil text** (the way Bouncer passes its rubric).
  Rejected: a seat stencil would have to know a fixed marker per block, and the board's "included from it by name" is exactly Go's `{{template}}`.
- **Transitive includes.** Rejected for a one-pass marker guarantee and no stencil needing it.
- **A mailbox of the producer's own** (files under the run directory, `lyx` verbs to post and read).
  Rejected: it is a second creel, which the board defers to creel itself; it would be built to be deleted.
- **`lyx shuttle send` as the seat channel.** Rejected: it needs an idle receiver and a one-line body, forcing a turn-taking protocol with file pointers, and every chair wait would be a held turn.
  It stays as the Go-side and operator path into the chair.
- **Advisors may message each other.** Rejected: the chair steers and loses visibility of peer chatter; no planned consumer needs it; the chair relays.
- **An advisor's death fails the step.** Rejected: a whole re-step for a helper the chair may not need; the chair is told and decides.
- **Detecting an advisor hang from the session-state model.** Rejected: an idle advisor waiting for questions is the normal state, so idle-without-output is not a stall here, and the model is in shadow mode by `shuttle-state`'s decision.
- **Inheriting live advisors when the chair is gone.** Rejected: the fresh chair has none of the dialogue they were part of, so they are stopped and every seat starts fresh.
- **Telling the chair of a start failure by `Send` once it starts.** Rejected: the send lands only at the chair's first idle turn boundary, while the failure is known before the chair's prompt is filled; the `failed_advisors` marker tells it once with nothing to retry.
- **Advisors without `KeepPane`, writing their output last.** Rejected: shuttle removes a Done run's strand, so the chair's dialogue with an advisor would end the moment it writes its file; `KeepPane` keeps it reachable until the chair's result stops it.
- **Advisors' parent directive naming the run's parent.** Rejected: it tells every advisor how to reach the orch and to trust only it, so the chair's messages would not be trusted and advisor traffic could reach the parent.
- **A shuttle probe form that hands back a handle for a terminal record over a live strand**, so a finished advisor stays reachable across a resume.
  Rejected: a second shuttle surface for a rare path, when the advisor's output is already on disk and the chair's hang rule covers a question that goes unanswered.
- **Probing advisors without superseded-strand removal on resume.** Rejected: shuttle answers not found for a terminal record and hands back no handle, so a kept advisor strand would outlive the chair's terminal result with nothing to stop it.
- **One goroutine per notice.** Rejected: one sender keeps the notices in the order their ends were observed and puts the stop rule in one place; the send lock alone would only stop them typing at once.
- **Re-reading the chair's outputs between `Run.Send`'s idle wait and its typing**, instead of a send lock. Rejected: the idle wait and the typing are one call inside shuttle, so the check would need a shuttle hook anyway, and it would only narrow the overlap with the gate's send, not close it.
- **Keeping a resume's stop of a finished advisor silent.** Rejected: the advisor block tells the chair a finished advisor keeps answering, so silence leaves the chair asking a seat that is gone until the hang rule writes it off; one line on the notice path already in use costs nothing.
- **Adding an advisor that ends during the chair's start to `failed_advisors`.** Rejected: whether it lands there would race the chair's prompt fill; the queued notice takes every end after a start down one path.
- **A second parent-directive variant for advisors**, naming the chair without the spawned-run clause. Rejected: a new stencil variant and render path for a clause the advisor acts on nothing in; the advisor block states the chair's real role.
- **Restarting a dead advisor on resume.** Rejected: its context is gone, the chair may have what it needs, and a restart invites a second copy if the probe misjudged.
- **Chair first, advisors after.** Rejected: under reed's current rule the newest strand takes the space, and the operator reads the chair; names are deterministic so order costs addressing nothing.
- **`SingleLLMProducer` becomes the N = 1 case.** Rejected: its result-shaped seam is shared with Bouncer and every test fake; collapsing it routes every loom LLM row through new code before any consumer exists.
- **Suppressing the chair's held-turn notice too.** Rejected: a chair that ends a turn without outputs may be stuck, and the parent should hear it as it does today.
- **Timeouts per seat.** Rejected: one step, one deadline; nothing asked for more.
- **A new PATTERN entry** such as "a multi-agent step's agents are declared in one seat table".
  Rejected for now: it would be false while burler's `roundHalves` exists, and `burler-multi-llm` is the task that could make it true.
- **A `lyx` verb to run a seat table standalone.** Rejected: the board asks for tests and a test recipe, not a product verb, and a verb brings a sandbox-suite obligation.

## Review rounds

_No rounds yet._

## Question ledger

Every entry is an auto-pick made under autonomous mode, with its reason; none was answered by an operator.

- **Q1. Where does the runner live?** Auto-pick: `internal/seatengine`, with `MultiLLMProducer` in `shedadapters` (D1). Reason: import direction for `burler-multi-llm`; adapters stay thin.
- **Q2. What is the channel?** Auto-pick: Claude Code's `SendMessage` by strand name (D6). Reason: delivery into a running turn fits the chair's ask-and-continue; a producer mailbox duplicates creel; `shuttle send` forces turn-taking.
- **Q3. May advisors message each other?** Auto-pick: no (D6). Reason: the chair steers; no consumer needs it.
- **Q4. What does an advisor's failure or hang do to the step?** Auto-pick: nothing to the outcome; the chair is told once; a hang is the chair's call (D7). Reason: the chair owns the outcome.
- **Q5. How does the producer know an advisor is done?** Auto-pick: its output files exist, the file contract; the session-state model is not read (D5, D7). Reason: `shuttle-state` keeps the model in shadow mode; idle is normal for an advisor.
- **Q6. How are the seats laid out?** Auto-pick: current rule, advisors first, chair last (D9). Reason: the chair gets the space; `reed-layout-shares` is separate.
- **Q7. Does `SingleLLMProducer` become N = 1?** Auto-pick: no; a test pins equivalence (D11). Reason: no behaviour gain for a shared, heavily tested seam.
- **Q8. How does a seat include shared blocks?** Auto-pick: `{{template "name"}}` added form, one level (D3). Reason: native to Go templates; keeps `stencil` a leaf.
- **Q9. Model per seat: a spec string or a config key?** Auto-pick: model-spec notation per seat in the recipe, resolved through `modelspec`; a Go caller passes the resolved triple (D2). Reason: every agent-spawning config uses the notation; a consumer maps its own `loom.yaml` keys in its own entry.
- **Q10. Interactive seats?** Auto-pick: the chair only (D10). Reason: an advisor's questions go to the chair, and the operator does not watch advisor panes.
- **Q11. Held-turn notices for seats?** Auto-pick: off for advisors, on for the chair (D10). Reason: advisor idling is routine; a chair idling may be stuck.
- **Q12. Seam for the recipe entry?** Auto-pick: a new nil-able `Env` seam, construction error when a row needs it and it is nil (D12). Reason: the `commit_seam` rule.
- **Q13. New PATTERN entry?** Auto-pick: none (Scope). Reason: nothing new is cross-cutting yet; a seat-table invariant becomes true only after `burler-multi-llm`.
- **Q14. Environment.** `lyx board get` and `lyx board list` worked; `lyx loom validate-discussion` is run at the end of this session; nothing outside the two output files was touched or found broken.
