# Orch rules for the loomyard hub

## Production

An exception to `CLAUDE.md`'s Production rule: after your own pushed fix, on a clean `main`, you run `update-plugins.sh` yourself and say so in your report.

## Watching runs

The operator watches each run in its own terminal window, never in VS Code.
VS Code opens only when the operator asks for it, through `lym ide` or `batten.yaml`'s `open_ide`.

## A review segment that already converged

When a Bouncer whose round already converged stops on something after the verdict, such as a failed commit, and its bounce budget runs out, never take the `goto` the error suggests: a goto gives the whole segment a fresh budget and runs a new review round over an approved artifact (`ubuntu-2404-support`, 2026-10-19).
Settle the stop with `lym loom circling accept <slug>` and `lym loom resume` in the task worktree, so the Bouncer settles on the verdict it has; when accept refuses, ask the operator before any goto.

## Only your own hub

You start runs only in this hub; a run in another hub, such as `quarry-LYMHUB`, is started by that hub's own orch.
Setting up another hub, or writing its board, happens only when the operator asks.

## No changes in the prime

Never edit, commit or push code or docs directly in the prime worktree: a change goes through a task worktree, so the prime is never dirty when a batten run starts or a run lands.
Board, config and orch-rules writes through `lym` are not prime edits.

## Say worktree

To the operator, a fabric checkout is one worktree: say "worktree", never "pair".

## No status polling

Never check runs with `lym batten status` on your own; runs are watched through their notices, and a command's own output says what it did.
When a status read is needed, filter it to the fields you need, never the full JSON with its history.

## The orch note only at a cycle

Write the orch note only when a message asks for it at a context cycle; never update it between cycles as a running log (operator, 2026-10-22).

## Dead strands after a reboot (#799)

After a machine restart, the strands from before it stay in `lym reed list` as dead and hold their names, so a resumed row can fail on a name clash.
When resuming runs after a restart, remove each dead strand of the run with `lym reed remove --name <strand>` in its task worktree, then `lym loom resume` a row that failed on the clash.
Remove this section once #799 is fixed.

## Loom only for bundles

Never propose one note alone as a loom task: a loom run has a fixed cost of about half its spend whatever its size (`docs/research/token-audit-2026-10-22`).
Fold related notes into one loom task, or propose darn for a small change, and let the operator name darn (operator, 2026-10-22).
Never propose splitting a piece out of a darn task as a loom run of its own: a part that looks large raises the darn's profile, it never becomes a loom (operator, 2026-10-26).

## Release only on a named go-ahead

Release a task, or take any other step that starts work, only on the operator's explicit go-ahead naming that step; never infer it from a rejected alternative or an answer to another question (operator, 2026-10-26).

## Board tidy after a landing

Dobby spawns the board tidy agent itself after landings; you never spawn it, count landings for it or edit the board for a tidy yourself (operator, 2026-10-26).
Dobby's line names the tidy's report path: a note removed whole, or a change of its scope or design, the report proposes, and you put it to the operator first.

## A decision for a run at Webster (#849)

Merriam refuses the notice `lym loom decision add` sends, since it comes from `ly:LOOM` and Merriam may not read the decision record.
While a run is at Webster, record the decision with the verb as usual, then message Merriam (`<shortname>:<slug>:webster`) yourself with the decision's content spelled out.
Remove this section once #849 is fixed.

## A decision for a working darn writer (#871)

A darn writer works in one long turn, and creel delivers the notice `lym loom decision add` sends only when the writer is idle.
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

Never choose `lym loom circling continue` on a review escalation without asking the operator: a new round is expensive (operator, 2026-10-25).
Always offer the cheaper path beside it: record the blocking fixes as decisions with `lym loom decision add`, then `lym loom circling accept`.

## Discussion rounds are not cut for a full body

Discussion explores the code, bounds the scope and weighs the impact; a full board body does not do that work, so it is no reason to lower a task's Discussion rounds (operator, 2026-10-25).

## Codex effort

Never give a Codex run `xhigh` effort or above, in a bench or anywhere else: it costs too much; `high` is the ceiling (operator, 2026-10-27).
A Codex reviewer always gets the same context a Claude reviewer of the same round gets, the PATTERN overview and the repository's agent instructions included (operator, 2026-10-27).
