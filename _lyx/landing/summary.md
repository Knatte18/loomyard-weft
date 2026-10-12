# Twill producer for Plan-Write and a fabric-vocabulary plan check

Plan-Write gains a `twill` producer beside `single`.
Under `twill` the plan writer forks itself once per lens of a four-lens fan before it writes the plan, then writes and self-checks the plan, marks it ready, takes the forks' findings in and rejects a BLOCKING or MEDIUM finding only with a cited code read.
The benches recommended this at `opus[medium]`: it repeated fewer Plan-Review and Webster findings than the single writer on the second reference task, at about twice the cost per plan.
A new deterministic check, `fabric-vocabulary-name`, refuses at `lyx loom validate-plan` and at dispatch a declared name the fabric-vocabulary scan would refuse.
It covers the declared test functions behind the vocabulary miss every bench arm shared and only the verify gate caught.

## What changes

- `loom.yaml` gains `plan_producer`, `single` or `twill`.
  An absent or empty value reads as `single`; any other value fails config load with an error naming both.
  With `single` the Plan-Write prompt is byte-identical to before and `ForkSubagents` stays false.
- Under `twill`, `PlanSpec` renders the twill directive into a new optional marker of `loom-template-plan` between Step 2 and Step 3 and sets `ForkSubagents`.
  The writer states the cards in a text reply and spawns the forks before it writes any file under the plan directory.
  The row, gates, output file, timeout, resume identity and plan-directory rotation are `single`'s.
- The embedded `burler.yaml` gains the `twill-plan` fan: `criteria-coverage`, `row-reachability`, `forced-edits` and `test-economy`.
  An unreadable `burler.yaml` or an unresolvable `twill-plan` fails the Plan-Write spawn with a refusal-spec row naming the fan, the file and the way forward.
  Discussion's fan resolution is shared, parameterised by fan and row name.
- The shared twill directive stencil takes three new host values: the spawn point, the rejection rule and the severity rule.
  The Discussion host fills them with its current text.
  The reviewed files are now one host-rendered phrase instead of a bullet list rendered inside backticks, which broke the boilerplate's "read the files under …" line; this is the only change to Discussion twill's rendered prompt.
- A plan ready marker, `00-overview.md.ready` under `.lyx`, is built in `internal/loomengine` beside the Discussion marker and removed before every fresh Plan-Write spawn, ahead of the plan-directory rotation.
  Attaching to a live writer or relaunching a dead one keeps it.
- `lens-review.md` in the plan directory records each finding with its lens and `fixed` or `rejected` with the citation.
  Its name is declared in `planparser`; the index check, the webster plan fingerprint and the prior-plan block ignore it, and rotation and the plan commit carry it with the generation.
  Its content is never parsed.
- `planparser` appends `fabric-vocabulary-name` to the plan index's format validation beside `card-fabric-reference`, so both plan gates, `validate-plan` and dispatch share it.
  It checks the member of every `Create` handle and every `Rename` pair's new handle, under any `language:`, and emits one blocking finding per name.
- The fabric-vocabulary predicates move out of `internal/lyxcwd/enforcement_test.go` into the new stdlib-only leaf package `internal/fabricvocab`, which `TestEnforcement_FabricVocabulary` and the new check both call.
  The package joins the vocabulary owner set and has its leaf row.

Closes: none; the board entry `twill-plan` lists no issues.

## Design decisions to review

- The check only refuses, and misses a name spelled only in a card's prose, such as a test row or subtest name, or built at implementation time; the verify gate still catches those.
- It may refuse a host-geometry helper declared in a `_test.go` file under `internal/`, which the scan admits; the rename it asks for is harmless.
- Its test-prefix heuristic misses a helper without a test prefix declared in a `_test.go` file outside `internal/`.
- The owner-set row exempts the whole `fabricvocab` package from the bare-token rule, so it must hold only the vocabulary predicates and their data and import only the standard library.
- Forks run on the writer's model, read-only and without the Agent tool.
  With `claude_deny_agent_tool` true the Agent hook admits only `subagent_type: "fork"`.
  A fork that gives up leaves the plan without its lens review, recorded in `lens-review.md`; Plan-Review and the plan gates still run.
- The fan name is fixed in `internal/loomengine`; a worktree that redefines `twill-plan` in its `burler.yaml` changes only the lens texts.
- PR-Rework's plan writer and Plan-Burler-Fix do not fork and are unchanged.

## To check or run by hand

- This repository does not select `twill` in this change.
  After landing and deploying, run `lyx config loom --set plan_producer=twill --set 'plan=opus[medium]'` in the hub's parent worktree; the prod binary refuses the key until the new binary is deployed.
- Run a real `plan_producer: twill` Plan-Write and confirm the forks spawn before any plan file, `00-overview.md.ready` is created, and `lens-review.md` is committed with the plan.
- `contracts/specs/loom-plan-spec.md` and the help text for `lyx loom validate-plan` were edited for the new check; read them against the code.
- The card and batch gates of the generating run listed `./internal/planparser_test`, which is not a package, so `lyx gate test` reported `[setup failed]` for it.
  Run `lyx gate test ./internal/planparser` and `lyx gate test --tags integration ./...` to confirm the package's tests pass.
- `config-layers-run-settings` is in flight and changes how `_lyx/config/*.yaml` layers resolve; the new `plan_producer` key may need rebasing onto its layer model.
- `seats-replaces-singlellm` and `plan-multi-llm` move Plan-Write off the single producer; each must carry the producer key, the ready marker and the lens-review file over.
