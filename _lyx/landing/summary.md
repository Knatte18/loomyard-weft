# Add twill, a self-forking Discussion-Write producer

`discussion_producer` in `loom.yaml` now accepts `twill` beside `single` and `seats`.
The writer explores and settles the design alone, then forks itself once per lens before it writes.
Each fork waits, keeping its prompt cache warm, until the record is ready, then reviews it through its lens as an outsider and returns findings.
The writer judges the findings, fixes the record or rejects them, and records each disposition in the last turn of the support log's `## Interview`.

The first Discussion bench found that seats advisors re-read the code at full price and did not improve the draft.
Forks run on the writer's model and read its exploration from cache, so they avoid that cost.

No issue is closed by this change.

## What changes

- **Config**: `twill` loads; any other value is refused with an error naming `single`, `seats` and `twill`.
- **Producer**: `twill` is a spec variant of the single writer, not a new engine.
  The row, gates, output files, timeout and resume identity are `single`'s.
  `DiscussionSpec` renders the twill block, resolves the fan and sets `ForkSubagents`.
  `single` renders a byte-identical prompt and keeps the full Agent deny; `seats` is untouched.
- **Template**: `loom-template-discussion` gains two optional markers, empty for `single`.
  One carries the twill directive between Steps 4 and 5.
  The other, below the "What you may write" list, widens the write scope by the ready marker and its parent directory.
  The commit sentence stays about the two discussion files.
- **Stencils and package**: `contracts/stencils/twill/` holds `twill-directive.md` (the fork-wait-review procedure) and `twill-write-scope.md`.
  `internal/twilldirective` renders both from the told stencils directory at call time.
  The stencils name no Discussion file, section or verb, so Plan-Write can reuse them.
- **Lenses**: the embedded burler template gains the fan `twill-discussion` (`premises`, `goal-scope`, `board-fit`, `pattern-fit`, `scenarios`, `leanness`) and the new `premises` and `board-fit` lenses.
  The fan name is fixed; redefining it in `burler.yaml` changes the lenses.
  An unresolvable fan fails the spawn with an error naming the fan and `burler.yaml`.
- **`lyx shuttle await <path>... [--cap]`**: a general capped wait that returns when any path exists (`ready: true`, naming it) or when the cap elapses (`ready: false`, exit 0).
  The default cap is 250 s, under the prompt-cache lifetime.
  It skips geometry, config and engine resolution, so it runs outside a hub and a git repository.
- **`lyx burler await-review`** keeps its name, argument and output, and now shares the poll, which moves from 200 ms to a one-second stat.
- **Docs**: `doc.go` for `internal/twilldirective`, `internal/shuttleengine`, and the `docs/overview.md` module table.

## Design decisions

- **Ready marker**: it lives under `.lyx`, its path built through `internal/lyxdirs`, and is written only after the writer's self-check passes, so forks never see a partial record.
  Go removes a stale marker before a fresh spawn; attaching to a live writer or relaunching a dead one keeps it.
  It is not an output file, so the completion check and gate list are unchanged.
- **One fork pass per writer spawn**: a gate re-prompt or a parent-review reject never forks again.
  A writer relaunched after its process died has lost its forks and resumes at its first unfinished step, recording the lost forks as "no report".
- **Fork bounds**: a fork gives up after 15 consecutive `ready: false` answers and makes at most 6 exploratory tool calls once the record is ready, reporting each.
  It is read-only and returns findings only as its final message.
  These bounds are stencil-level; the hook admits a fork spawn from any session in the run.
- **Fork authorization**: with `claude_deny_agent_tool` true, the edge admits only `subagent_type: "fork"`.
  With it false, no Agent hook is installed, as for `single`.
  No Go-side audit of fork writes is added; the downstream gates read the two discussion files only.
- **Wait semantics**: an incoming SendMessage is not a wait condition, since lyx cannot see Claude Code's inbox.
  A sender that needs an immediate wake also writes a marker the waiter watches.

## To check by hand

- The bench of `twill` against solo `opus[medium]` and `opus[high]` has not run; it follows after landing.
- Nothing here was run against a live Claude Code session.
  Check that forks inherit the writer's context, report completion, and surface as outstanding work as the wait assumes.
- Six forks may cost more than they find; compare cost against the findings they raise.
- A lens review longer than the 5-minute cache lifetime wakes the writer on a cold cache; nothing keeps it warm.
- `internal/burlercli/await.go` and `contracts/stencils/claims_test.go`: confirm the fixer stencil's claim for `await-review` still holds after the move to the shared one-second poll.
- `contracts/stencils/loom/loom-template-discussion.md`: confirm the widened write scope and do-not-repair rule read correctly when rendered for `twill`, and that the commit sentence still names only the two discussion files.
