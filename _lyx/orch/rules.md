# Orch rules for the loomyard hub

## Production

An exception to `CLAUDE.md`'s Production rule: after your own pushed fix, on a clean `main`, you run `update-plugins.sh` yourself and say so in your report.

## Watching runs

The operator watches each run in its own terminal window, never in VS Code.
VS Code opens only when the operator asks for it, through `lyx ide` or `batten.yaml`'s `open_ide`.

## A review segment that already converged

When a Bouncer whose round already converged stops on something after the verdict, such as a failed commit, and its bounce budget runs out, never take the `goto` the error suggests: a goto gives the whole segment a fresh budget and runs a new review round over an approved artifact (`ubuntu-2404-support`, 2026-10-19).
Settle the stop with `lyx loom circling accept <slug>` and `lyx loom resume` in the task worktree, so the Bouncer settles on the verdict it has; when accept refuses, ask the operator before any goto.

## Only your own hub

You start runs only in this hub; a run in another hub, such as `quarry-LYXHUB`, is started by that hub's own orch.
Setting up another hub, or writing its board, happens only when the operator asks.

## No changes in the prime

Never edit, commit or push code or docs directly in the prime worktree: a change goes through a task worktree, so the prime is never dirty when a batten run starts or a run lands.
Board, config and orch-rules writes through `lyx` are not prime edits.

## Say worktree

To the operator, a fabric checkout is one worktree: say "worktree", never "pair".

## No status polling

Never check runs with `lyx batten status` on your own; runs are watched through their notices, and a command's own output says what it did.
When a status read is needed, filter it to the fields you need, never the full JSON with its history.

## The orch note only at a cycle

Write the orch note only when a message asks for it at a context cycle; never update it between cycles as a running log (operator, 2026-10-22).

## Dead strands after a reboot (#799)

After a machine restart, the strands from before it stay in `lyx reed list` as dead and hold their names, so a resumed row can fail on a name clash.
When resuming runs after a restart, remove each dead strand of the run with `lyx reed remove --name <strand>` in its task worktree, then `lyx loom resume` a row that failed on the clash.
Remove this section once #799 is fixed.

## Loom only for bundles

Never propose one note alone as a loom task: a loom run has a fixed cost of about half its spend whatever its size (`docs/research/token-audit-2026-10-22`).
Fold related notes into one loom task, or propose darn for a small change, and let the operator name darn (operator, 2026-10-22).

## Board tidy after a landing

Once `loom-tail-review` has landed, PR-Align no longer names the board entries a run superseded, so at every fifth `landed` notice you spawn one fresh Sonnet agent, never a fork, with the operator's standing go-ahead (operator, 2026-10-23; every fifth, 2026-10-24).
Keep the landings since the last tidy in the orch note, and give the agent all of them at once.
It runs as `model: sonnet` with effort `medium`, never Opus and never a higher effort: the job is cheap reading (operator, 2026-10-24).
Its brief: read each landed commit and its run's decision record against the board's notes, through `lyx board` only; make each obvious amendment itself, such as dropping a sentence the landing made stale or noting that part of a note is now built (body drafts only in `<prime>/.scratch/`); and report what it changed, plus each note the landing superseded whole or changed in scope or design, with why.
You never edit the board for a tidy yourself: the agent does it (operator, 2026-10-24).
A note removed whole, or a change of its scope or design, you put to the operator first, and have the agent carry out the operator's answer.
This job moves to the orch's Go driver once there is one, which spawns the same agent every N landings, N a config key with 5 as its default.

## A decision for a run at Webster (#849)

Merriam refuses the notice `lyx loom decision add` sends, since it comes from `ly:LOOM` and Merriam may not read the decision record.
While a run is at Webster, record the decision with the verb as usual, then message Merriam (`<shortname>:<slug>:webster`) yourself with the decision's content spelled out.
Remove this section once #849 is fixed.

## A decision for a working darn writer (#871)

A darn writer works in one long turn, and creel delivers the notice `lyx loom decision add` sends only when the writer is idle.
After recording a decision for a darn run whose writer is working, also send the decision's content by `SendMessage` to `<shortname>:<slug>:darn`.
Remove this section once #871 is fixed.

## A board body holds the current design only

When a newer design, decision or answer replaces part of an entry's body, rewrite the body so it holds only what stands now: remove the replaced drafts, recommendations and open questions in the same write, never keep them under a "this replaces everything above" line (operator, 2026-10-25).
Bolt's git history keeps the old text.

## Explorations run as darn docs

A bench, spike or other exploration the operator approves runs as a darn task with profile `docs`, in its own worktree, so it is not disturbed and its report lands in `docs/research/` as a commit (operator, 2026-10-25).
Never run such work as a background agent of your own; an agent of yours only reads and answers within your turn's scope, such as a quick study for a chat question or the board tidy.
The operator naming an exploration for a run counts as naming darn for it.
Its body states an exact cap on runs: inputs, models, efforts and repeats by number, one or two inputs unless the operator names more, and never an open word such as "many" (operator, 2026-10-25, after a Codex bench of 31 runs spent the plan's 5-hour window).

## A review escalation asks before a new round

Never choose `lyx loom circling continue` on a review escalation without asking the operator: a new round is expensive (operator, 2026-10-25).
Always offer the cheaper path beside it: record the blocking fixes as decisions with `lyx loom decision add`, then `lyx loom circling accept`.

## Discussion rounds are not cut for a full body

Discussion explores the code, bounds the scope and weighs the impact; a full board body does not do that work, so it is no reason to lower a task's Discussion rounds (operator, 2026-10-25).

## A card slug ending in -weft (#874)

Webster's fabric-reference audit flags every unquoted word ending in `-weft`, so a card whose slug ends that way fails its batch audit whenever a fork names the card in a command.
Judge such a finding false when the matched commands only name the card, have Merriam run `lyx webster accept-audit --batch <n>` and `recover-batch <n>`, and have later forks quote the slug.
Remove this section once #874 is fixed.
