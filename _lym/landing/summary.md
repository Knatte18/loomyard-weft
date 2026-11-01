# sampler: an interactive loom recipe the operator iterates in

Adds `sampler`, a loom recipe for work that takes many rounds between the operator and an agent on the result itself: a README written as a teaching text, a board entry, a trial of something visible in tmux.
Darn's writer is non-interactive and tapestry's `discussion_interactive` only interviews the operator before anything is written, so neither fits.

The build segment is one interactive agent in the run's own worktree.
It waits for the operator in its pane, commits as it goes, and ends the segment with the new verb `lym loom sampler-done` when the operator says the work is done.
The run then goes on as any loom run does: Code-Review perch (off by default), Describe, PR-Review, Finalize, Friction-Reflect.
While the agent waits, dobby's quiet notice and the step watchdog are off.

## What changes

- **Recipe.** A `sampler` recipe file with the `Sampler-Session` build row, registered as a new `SamplerSession` engine in `internal/shedrecipe` and in `LoomBuildEngines`.
  The recipe has no Drift or Parent-Align rows; Finalize merges the parent under the parent lock, as darn's does.
  `Describe` always runs, because the sampler agent writes no description and Finalize parses one first.
- **Vocabulary.** `sampler` is accepted as a seed recipe (`internal/shedrun`), armed by `lym shed` (`internal/shedcli`), accepted as a board task's `recipe` (refused on a project, as `darn` is), and as a block in run profiles (`internal/runprofile`).
- **Config.** A hub-wide `sampler.yaml` module in the new `internal/samplerengine`, with `agent` (default `opus[high]`) and `code_review` (default `false`).
  A task turns the perch on with `run_settings` `{"sampler":{"code_review":true}}` or a profile block.
  Finalize's perch-off readings (`rerouteReviewed`, `codeBudgetZero`) now follow the perch rows' `entry_gate`, so a reroute on a sampler run with the perch off halts for the orch.
- **Session.** `internal/loomshed/samplersession.go` keeps one interactive session per run, started unawaited as PR-Review's is: a live strand is reused, a recorded session id whose strand is gone is resumed, otherwise a new session starts.
  The `sampler-template-agent` stencil tells the agent to read the record as its brief, work only in the worktree, commit per settled step, write board results as notes only, and run `sampler-done` only when the operator says so.
- **The wait and `lym loom sampler-done`.** The row waits with no timeout on a done record in the run's untracked scratch.
  The record carries an entry stamp, so a flag written before a `goto` back to the row is ignored; a halt and resume leave the stamp unchanged.
  The verb refuses on a recipe without the row, on another current row or before a session is recorded, and on a dirty code checkout, each with a way forward.
  A dirty checkout found at the wake clears the flag and mails the agent the reason through creel; a damaged record is Stuck with Halt naming its restore steps.
  The verb is accepted while the run is halted at the row, so the next `lym loom resume` ends the row Done without starting a session.
- **Decision records.** `lym loom decision add` admits one more caller: the strand the run's `Sampler-Session` row has recorded, only with `--by operator`.
  The run's record is written from the board entry, as darn's is.
- **Operator-wait rows.** One list, `OperatorWaitRows` / `WaitsOnOperator` in `internal/loomshed`, names `Beam-Await`, `PR-Review` and `Sampler-Session`.
  Dobby's `childAtWaitingRow` and the step watchdog's `agentActivityFor` read it.
- **Loom wiring.** The verify, budget, record and tree-scope seams that were keyed on a `DarnWrite` row are keyed on the rows that need them, so a sampler run's Code-Burler-Fix gets the same verify gate as darn's.
- **Docs.** The pinned specs, package docs, `PATTERN-daemon-wakeups` and `docs/overview.md` lines that name the loom recipes.

## Design decisions to know

- **Trials.** No new tmux mechanism. The stencil tells the agent never to run a dev build of lym in this hub, since any reed operation re-pins the hub's tmux server-wide key bindings, to try it in the sandbox hub (`docs/sandbox-howto.md`), and to start any other tmux on its own `tmux -L <name>` server and kill it before `sampler-done`. The stencil is the only guard.
- **Decision label.** The `--by operator` label is unverified, so a sampler agent could record a decision the operator did not settle. Entries stay append-only, labelled, dated and committed, so the operator can read every one.
- **Watchdog.** A wedged step at a listed operator-wait row is no longer ended by the step watchdog, only by `lym batten cancel` or the row's own halts.
- **No build-row verify gate.** Finalize's gate over the merged tree checks any code a sampler run lands.
- **Batten** is unchanged for every recipe, including Open-Window and dobby's start rule.

## To check by hand

Not run: tiers 2 and 3 of the gate on the individual changes; the final gate result is the reviewer's to confirm.

- An existing hub's `profiles.yaml` is seed-only and gains no `sampler` block. A sampler task naming a profile there is refused at seed until the orch adds one with `lym config profiles`.
- A real sampler run end to end in a tmux pane: the agent waiting, an operator round, `lym loom sampler-done`, then Describe through Land. Also a board-note-only run landing empty with no pull request.
- The trial rule rests on one sandbox hub per machine, so two trials on one machine share it. Confirm this is what was meant by "a tmux server of its own", or whether lym should give each trial run a server.
- The wait's done-flag branches on the wake path (dirty checkout clears the flag, damaged record halts) were the one untested area a review found; confirm the added rows in `internal/loomshed/samplersession_test.go` pin them.
- The stencil cites a pattern by bare name in one place; check that `internal/samplerengine`'s stencil reads sensibly to the agent without the pattern file.
- Test names for the Sampler-Session row are pinned in the acceptance list; confirm each acceptance criterion has a test named for it.
- The verify gate once failed `TestGofmt_ModuleFilesAreFormatted` because the `lym config` long text passed its byte cap once the `sampler` module name was added. It was fixed by shortening that text (`13f6dd08`); check the shortened help still reads well.
- The producer's halt hint for `lym batten cancel` could carry a literal `<slug>` placeholder, since `SamplerDeps` has no slug field; check the rendered hint.
