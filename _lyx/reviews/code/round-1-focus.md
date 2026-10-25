---
round: 1
focus:
  - "internal/dobbyengine/view.go and starts.go: does BuildView share the exact predicate and max_runs counting of readyToStart and runsInFlight, or restate them? Can the view list a wait the scheduler lacks, or miss one it has?"
  - "internal/dobbyengine/starts.go and state.go: is the refusal text written by the refused start and cleared when a pass claims the task or its entry changes, through UpdateState, with the key optional?"
  - "internal/battencli/dobbyreads.go against the removed internal/beamcli/dobbydeps.go: is the moved read logic unchanged except for the decided change that parent and owner are read for every task with an empty status, and are TaskView's open dependencies by slug correct?"
  - "internal/battencli/overview.go and overviewdobby.go: does the dobby section appear only in the prime and beam worktrees, never in a task pair's frame, below the run lines after a blank line? Do times render in minutes so the poll keeps backing off, and does the section add no wake?"
  - "internal/beamcli/cli.go: is `lyx beam status` a cobra subcommand of the beam tree that covers the prime and every local beam as `lyx beam start` considers them, with the view in its envelope?"
  - "internal/dobbyengine/loop.go and passes.go: does the Log seam write one line per task start and per pass spawn with time, kind, slug and pass kind, do nothing when nil, and is the hub wiring pointed at stdout?"
  - "dobbyengine import allowlist test and PATTERN-cwd-resolution: does dobbyengine still import no lyxcwd or CLI package, and was the allowlist test updated for any new import?"
  - "Docs in the same diff: doc.go of dobbyengine and beamcli, docs/overview.md, and the stencil edit in contracts/stencils/orch/orch-template-role.md. Does the stencil change drop or weaken a rule the board entry did not ask to change, and does the diff need a PATTERN.md entry?"
  - "Test economy: do view_test.go, overviewdobby_test.go, status_integration_test.go, passes_test.go and starts_test.go duplicate each other's coverage, and is any added behavior uncovered (the refusal text clearing, the pass in flight, the ready case)?"
  - "The commit message and landing summary: does each claim, such as the reasons listed and one log line per start and per pass, hold in the diff?"
---

Seed focus for round 1 from the board entry and the decision record's scope, decisions and acceptance criteria.
