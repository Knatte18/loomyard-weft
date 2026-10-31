# Decision record — interactive-recipe

## Goal

Add `sampler`, a loom recipe for work that takes many rounds between the operator and an agent on the result itself: a README written as a teaching text, a board entry, a trial of something visible in tmux.
Its build segment is one interactive agent in the run's own worktree that waits for the operator in its pane, iterates with them, commits as it goes, and ends the build segment through a `lym loom` verb when the operator says the work is done.
The run then goes on as any loom run does: the Code-Review perch, off by default, Describe, PR-Review when the landing wants a pull request, Finalize and Land.
While the agent waits for the operator, dobby's quiet notice and every idle timeout are off.

## Scope

In:

- The `sampler` recipe file, its rows and its place in the recipe vocabulary, the board's `recipe` field and the run profiles.
- A hub-wide `sampler` config module.
- The `Sampler-Session` row, its interactive session, its stencil and its wait.
- The `lym loom sampler-done` verb.
- The caller rule of `lym loom decision add`, widened for a sampler run's own agent.
- The one list of rows that wait on the operator, read by dobby's quiet notice and by the loom loop's step watchdog.
- The loom wiring that today keys darn's verify, budget and record seams on the `DarnWrite` engine.
- The pinned specs, package docs and `docs/overview.md` lines that name the loom recipes.

Out:

- Batten: its rows, its Open-Window row and dobby's start rule stay as they are for every recipe.
- Parking an idle interactive run (note `interactive-idle-park`).
- Waking PR-Review's wait at once on `pr-continue` (note `pr-continue-wakes-row`).
- A new tmux isolation mechanism for trials (D9).
- Letting a run's agent create or promote a board task.

## Decisions

### D1 — The recipe's rows

`contracts/recipes/sampler-recipe.yaml` (new) declares `contract: loom` and `verify_source: tree_scope`, with `entry: Preflight` and `Friction-Reflect` as its terminal.
Its `on_done` path is `Preflight` → `Record-Write` → `Sampler-Session` → `Code-Bouncer` → `Describe` → `PR-Review` → `Finalize` → `Friction-Reflect`.
`Code-Bouncer`, `Code-Burler-Review` and `Code-Burler-Fix` form the Code-Review segment exactly as in `contracts/recipes/darn-recipe.yaml`, each carrying `entry_gate: sampler.code_review`.
`Describe` carries no `entry_gate` and always runs, with darn's description gate, because the sampler agent writes no change description and Finalize parses one first.
`Finalize` keeps `on_reroute: Code-Bouncer`.
The recipe has no Drift rows and no Parent-Align row: the work is prose, board entries and trials, and Finalize merges the parent itself under the parent lock, as darn's does.
`SamplerSession` is a new engine in the one registry map of `internal/shedrecipe/registry.go`, and joins `LoomBuildEngines`, so the loom contract's build-row clause holds.
`verify_source: tree_scope` reads the hub's `tree_scope` from `darn.yaml`, so a diff wholly under the trees it lists runs only those trees' targets at Finalize; a hub that lists none runs the full landing gate.

Rationale: the operator's sequence is record, interactive build, Code-Review perch, Describe, Finalize and Land; PR-Review stays because only its resolver decides whether a landing goes through a pull request (`PATTERN-pull-request-mode`), and it passes with no pull request when the resolver says no, as for an empty landing.

### D2 — The recipe's name reaches every recipe vocabulary

- `internal/shedrun/seed.go` gains `RecipeSampler` in its closed vocabulary, and `internal/shedcli/table.go` gains a `sampler` entry armed by `loomcli.ArmAt` with the same verbs as darn.
- The board's `recipe` field accepts `sampler` on a task, and refuses it on a project as it refuses `darn`.
- `internal/runprofile` accepts a `sampler` block in a profile.
  The template's `speed`, `light`, `solid` and `wide` profiles each gain a `sampler` block carrying their `darn` block's `loom` perch values; `speed`'s block leaves `sampler.code_review` false, and `light`'s, `solid`'s and `wide`'s set it true, so naming one of those three asks for the review.
  The template adds no `defaults` entry for `sampler`, so a task naming no profile runs on the hub's own config, with the perch off.

Rationale: the recipe name is a closed vocabulary with one declarer and tables kept in step by their tests; a profile without a block for the task's recipe is refused at seed, so the template's ladder profiles must carry one.

### D3 — A hub-wide `sampler` config module, and one perch switch

A new package `internal/samplerengine/` (new) owns `sampler.yaml`, registered in `configreg` like darn's module and loaded only through `configengine`.
Its keys:

- `agent`: the model-spec of the sampler agent's session, default `opus[high]`, validated at load.
- `code_review`: whether the Code-Review perch runs, default `false`.

A task turns the perch on with `run_settings` `{"sampler":{"code_review":true}}` or a profile block carrying the same; the perch then reads loom's `code_*` keys as darn's does.
Every reading of whether the Code-Review perch is off on a run follows the perch rows' `entry_gate` as well as the segment budget: Finalize's `rerouteReviewed` and `codeBudgetZero` and the landing's `CodeReviewOff` in `internal/loomcli/landingdeps.go` read a perch whose rows the gate leaves out as off, so a Finalize reroute on a sampler run with `code_review` false halts for the orch as on any run with the perch off.
The package also builds the agent's session spec from told paths and values, as `internal/darnengine/` builds the darn writer's table, and imports no resolver.

Rationale: the template default reaches every hub through the sparse hub base.

### D4 — The record and the agent's decisions

`Record-Write` writes the sampler run's decision record and support log from the board entry, as it does for darn: the body verbatim under `## Goal`, the other sections empty, both committed.
What the operator and the agent settle on the way, the agent appends with `lym loom decision add --by operator`, so Describe and the Code-Review perch measure against the amended record.
The caller rule in `internal/loomcli/decision.go` admits one more caller: the strand the target run's `Sampler-Session` row has recorded as its session at the time of the call, while the run's current row is that row, and only with `--by operator`.
The row re-records the strand on every resume or restart of the session, so the admitted strand is always the live one.
Any other label from that strand, and that strand once the row has ended, are refused with a way forward naming `--by operator` or the run's parent.
`--amends-plan` refuses on a sampler run by its rows, as on darn.

Bound: the label is still unverified, as the caller rule's own doc says, so a sampler agent can record a decision the operator did not settle; what bounds it is that the operator watches the pane, the stencil allows only what the operator settled, and Describe shows every added entry.

### D5 — The `Sampler-Session` row starts one interactive session

The row keeps one interactive agent session per run, in a window of its own in the task worktree, started through shuttle's unawaited session start as PR-Review's session is (`internal/landingshed/prsession.go`):

- A recorded strand that is live is reused, a recorded session id whose strand is not live is resumed, and otherwise a new session starts; each resume or start re-records the strand's name, guid and session id.
- The session starts from a new stencil, `sampler-template-agent`, read through `stencilstore` at call time and filled with paths and values only: the slug, the decision record, the worktree.
- It runs under `sampler.agent`, loads `scribe:prose` and `scribe:conversation`, carries the build segment's colour and the role constant `sampler` from `internal/samplerengine/` (new), and renders the parent directive in its interactive form and the edit directive (`PATTERN-parent-directive`, `PATTERN-agent-name`).
- Its pane process is registered under the run (`PATTERN-run-process-owner`), and it gets the publish shell guard PR-Review's session gets.
- A start that fails transiently is an error the driver re-steps; any other start failure is Stuck with Halt naming `lym loom resume`.

No shuttle deadline applies, because nothing awaits the session: an unawaited start has no timeout classification.

Rationale: PR-Review's session is the existing shape of an agent that talks with the operator for as long as the operator wants while a Go row waits; sharing it keeps one way of keeping, resuming and stopping such a session.

### D6 — The row's wait and `lym loom sampler-done`

The row waits in process with no timeout, on a done record under the run's never-tracked scratch, named through `internal/lymdirs` and written through `internal/state` (`PATTERN-durable-vs-ephemeral-state`, `PATTERN-tolerant-records`).

- The record carries an entry stamp: the number of Done entries the run's status history holds for the row.
  A halt and a resume leave it unchanged, and a return to the row after it ended Done, by Finalize's reroute or a `goto`, raises it.
- `lym loom sampler-done` writes the done flag beside the stamp it computes.
  It refuses, each with a way forward and a test reaching it (`PATTERN-refusal-way-forward`):
  - a run whose recipe has no `SamplerSession` row, judged by its rows (`PATTERN-loom-contract`);
  - a run whose current row is not that row, or whose row has recorded no session yet;
  - a code checkout with an uncommitted change, naming a commit as the way forward.
- The wait wakes at least a second apart, doubling to a minute while nothing changes, and a file event on the record cuts a sleep short, so the verb takes effect at once (`PATTERN-daemon-wakeups`).
- A wake covered by a pause stops the session if it is running and judges nothing else.
  The first uncovered wake after a covered one resumes the session before any other check, as PR-Review's does.
- An uncovered wake that finds the recorded strand not live, when the row did not stop it, is Stuck with Halt naming `lym loom resume`, which re-enters the row and resumes the recorded session.
- A wake that reads a done flag carrying the current stamp checks the code checkout and the record:
  - a clean checkout and an intact record stop the session and end the row Done;
  - an uncommitted change clears the flag, mails the session the reason through creel and keeps waiting;
  - a record that fails the check Darn-Write's post-session guard runs is Stuck with Halt naming its restore steps.
  A flag carrying an older stamp is ignored.
- The record check and the verify, budget and tree-scope seams that `internal/loomcli/darnwiring.go` fills today only for a recipe with a `DarnWrite` row are wired for every recipe whose rows need them: a Record-Write record row, a Code-Review segment, `verify_source: tree_scope`.

Bound: the verb is the only way the build segment ends besides a halt or `lym batten cancel`.
It is not limited to the agent's strand, so the operator's own shell may run it too, and it is accepted while the run is halted at the row, so the next `lym loom resume` ends the row Done at once on a clean checkout.
A dirty checkout never ends the row, so Describe and Finalize always start from a committed tree.
An operator who closes the agent's window to stop gets a halt, and the stop chain or `lym reed resume` may reopen the session; ending the run for good is `lym batten cancel`.

### D7 — One list of rows that wait on the operator

The rows a run's child may wait in with no agent working are named in one list: `Beam-Await`, `PR-Review` and `Sampler-Session`.
Dobby's wait-live reading in `internal/battencli/wire.go`, which today names the first two, reads the list, so a child at `Sampler-Session` sends no quiet notice.
The loom loop's step-watchdog activity reading in `internal/loomcli/looparm.go` reads the same list and reports activity now while the child is at a listed row, so the watchdog never kills a step waiting on the operator.

Bound: a wedged step at a listed row is no longer ended by the watchdog; it is ended only by `lym batten cancel` or the row's own halts, as batten's Run-Shed step already is.

### D8 — What the agent's stencil tells it

The `sampler-template-agent` stencil binds the agent to make the result with the operator, never to write a plan or a record of its own:

- Read the decision record as the task's brief, say in a line or two what it understood, and wait for the operator.
- Work only in the run's worktree; commit as it goes, with a commit per settled step.
- Write a board result through `lym board`, as a note; a run's agent may never create or promote a task or change its own run's entry, which the board refuses, and the operator promotes a note when they want one.
- Record each thing the operator settles that changes the task's scope with `lym loom decision add --by operator`.
- Run `lym loom sampler-done` only when the operator says the work is done, after committing.
- Follow D9 for a trial.

Rationale: the board entry's own description of the agent.

### D9 — A trial runs outside this hub's tmux server

A trial needs no new lym mechanism.
Reed runs one tmux server per hub, named from the hub's path by `reedengine.ServerName`, and every reed operation re-pins the server-wide key bindings from a newer build, so any dev build of lym run in this hub can change the operator's own server.
A different hub has a server of its own.

The stencil tells the agent:

- Never run a dev build of lym in this hub; try it only in another hub on the machine, on lym's own repository the sandbox hub that `docs/sandbox-howto.md` describes.
- Start any tmux the trial needs outside lym on a server of its own, `tmux -L <name>` with a name holding the run's slug, and kill that server before running `lym loom sampler-done`.
- Commit what the trial settled, such as code or config, as an ordinary change.

Bound: the agent's adherence to the stencil is the only guard; nothing in lym stops a dev build run in this hub.

## Constraints

- `PATTERN-loom-contract`: the recipe declares `contract: loom`, and `sampler-done` and `decision add`'s new caller rule judge a run by its rows, never its recipe's name.
- `PATTERN-shed-recipe-registry`: `SamplerSession` is one entry in the one `Constructor` map, and the perch rows are left out by their `entry_gate`, never by a switch in a producer.
- `PATTERN-interactive-spawn` and `PATTERN-shuttle-provider-seam`: the agent is an interactive tmux session started through shuttle, its model-spec through `NewModelChoice`.
- `PATTERN-stencil-ownership`: the stencil is read at call time.
- `PATTERN-config-layers` and `PATTERN-sparse-hub-base`: `sampler.yaml` is loaded only through `configengine`, and a hub base holds only what the hub tuned.
- `PATTERN-dobby-starts-runs`: unchanged; dobby starts a sampler run as any other.
- `PATTERN-test-speed`: each test in its lowest fitting tier, run through `lym gate test`.
- Fact, from the code at this branch's base: a run's agent may write notes and never a task, from `internal/boardengine/doc.go`'s "Run agents file notes, never tasks"; a run with no commits beyond its parent lands with no pull request, from commit `5f444b56`.
- Fact, from the code: the Describe description gate checks only the file's format, a non-empty file, a `# ` title and a non-empty body (`internal/summaryparser/description.go`), never the diff, so a description of an empty diff passes it.
- Fact, from the code: `decision add` refuses every task session but the run's parent orch and its arbiter strand, from `internal/loomcli/decision.go`'s package comment, which D4 widens.
- Fact, from the code: the step watchdog's activity reading counts a shuttle wait only when it is not a held turn end, and an interactive session waiting on the operator is held, so a row waiting in process with no exemption is killed after `step_idle_timeout_min` (`internal/loomengine/template.yaml`, default 60); D7's list exempts `Beam-Await` and `PR-Review` from it too.

## Auto-mode assumptions

- Batten opens the window only through its existing Open-Window row and `batten.open_window`; a hub that turned it off attaches with `lym reed attach` or `lym reed window`, since the entry says batten stays the same for every recipe.
- "A run is started only when the operator means to work with it" is read as the reason for D7, not as a new start rule: releasing the task is the operator's go-ahead, and dobby starts it as any other.
- The agent's default model is `opus[high]`, since it works in conversation with the operator on prose and design.
- The build row runs no verify gate; Finalize's gate over the merged tree checks any code a sampler run lands.

## Open risks

- For the run's parent: the entry says a trial run "can use a tmux server of its own". D9 meets it with another hub, the machine's shared sandbox hub on lym's repository, and a stencil rule, with no per-run server in lym. Is that what the operator meant, or should lym give each trial run a server of its own?
- D7 changes the watchdog's reading for `Beam-Await` and `PR-Review` as well. If those rows were in fact protected some other way, the change is a no-op for them; if they were not, long PR reviews were being killed, which this fixes. Neither was checked in a live run.
- An existing hub's `profiles.yaml` is seed-only and gains no `sampler` block; a sampler task naming a profile there is refused at seed until the orch adds one with `lym config profiles`. A check by hand belongs in the change description.
- The trial rule rests on a sandbox hub the operator keeps per machine; two trials on one machine share it.

## Acceptance criteria

- D1: the sampler recipe builds and passes the loom contract check; its `on_done` path runs Preflight, Record-Write, Sampler-Session, the Code-Review segment, Describe, PR-Review, Finalize and Friction-Reflect in that order.
- D1, D2, D3: with `code_review` false the three Code-Review rows are recorded Skipped and spawn nothing, and Describe still runs and writes the description, also under a Code-Review budget of 0 such as `speed`'s; with it true the perch runs with loom's `code_*` values.
- D3: on a sampler run with `code_review` false and a non-zero hub budget, a Finalize reroute reads the perch as off and halts for the orch rather than rerouting past skipped rows.
- D2: a seed for `sampler` is accepted, `lym shed` arms it, a board task with `recipe: sampler` is accepted and a project with it refused, a task naming `light` seeds with `code_review` true, and a task naming no profile runs with the perch off.
- D3: `sampler.yaml` loads from its template, refuses an unresolvable `agent`, and reaches a hub with no file of its own.
- D4: on a sampler run at Sampler-Session, `decision add --by operator` from the row's recorded strand appends and commits an entry, also after the session was resumed under a new strand; another label from that strand, that strand after the row ended, and another task strand are refused with their way forward.
- D5: the row starts a session on first entry, reuses a live strand, resumes a recorded session id whose strand is gone and re-records it, and a non-transient start failure is Stuck with Halt naming `lym loom resume`.
- D6: `sampler-done` refuses on a darn run, on a sampler run at another row, before a session is recorded, and on a dirty code checkout; on a clean checkout it ends the row Done and the session is stopped.
- D6: a done flag written before a reroute or `goto` back to the row is ignored after it; a flag written before a halt ends the row at the resume; a flag read with a dirty checkout is cleared and the session is mailed; a damaged record at the flag is Stuck with Halt; a strand that died unstopped is Stuck with Halt naming `lym loom resume`.
- D6: a covering pause stops the session, and the first uncovered wake resumes it with no halt.
- D6: the verb's write ends the wait within a second, with no wait for the backed-off timer.
- D6: on a sampler run with `code_review` true, the Code-Burler-Fix verify gate gets the tree scope and `verify_attempts` budget a darn run's gets, and the row's record check is wired.
- D7: a child at Sampler-Session, PR-Review or Beam-Await reads as a live wait to dobby and as activity now to the step watchdog; a child at any other row reads as before.
- D8: the rendered stencil carries the parent directive in its interactive form, the edit directive, the decision record path and the slug, and names `sampler-done`, `decision add --by operator` and notes-only board writes.
- D9: the stencil names the no-dev-build-in-this-hub rule and the own `tmux -L` server rule for a trial, and names no denied form.
- D1, D6: a sampler run whose agent wrote only a board note, with no code commit, passes Describe's gate and lands with no pull request through the existing empty-landing path.

## Notes for the plan writer

- `internal/landingshed/prsession.go`'s `ensureSession` and its stop and resume around reads are the model for D5 and D6; sharing that code between the two rows, or copying its shape, is the plan's call.
- `childAtWaitingRow` in `internal/battencli/wire.go` is the list D7 replaces; the list must live where both `internal/battencli` and `internal/loomcli` can import it, and outside `internal/boardengine`, which imports no recipe package.
