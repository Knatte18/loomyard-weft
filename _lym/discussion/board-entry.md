# Board entry `codex-astra-tuned-prompt-bench`: Bench the forked Codex Astra fan with a Codex-tuned reviewer prompt

Context parity plus a prompt block asking for code, PATTERN-background and test reads, Astra high, on P4

## Question

Does a reviewer prompt tuned for Codex make Codex Astra's forked lens fan find the Claude fan's findings on a plan review, given the same context the Claude reviewers had?
The Claude fan's three `BLOCKING` findings on `P4`, which no Codex run has found (`docs/research/codex-astra-fork-fan-bench-2026-10-27/report.md`), each need a read beyond the plan's cards: the owner guard on `retryWeftPush` reading origin's Bolt outside `ReconcileOrigin`, the missing `adoptcli` entry in `PATTERN-cli-cobra`'s background, and the one-second production wait in `TestSetOwners`.

## Cap

- One input: `P4`, the `machine-handover` plan, with the forked fan bench's snapshot, lenses and grading.
- One model and effort: `gpt-6-astra` at high effort for the handler and its forks; never `xhigh` or above.
- One forked fan run: one `codex exec` handler spawning 3 lens forks with `fork_turns="all"`, no repeat, no Claude run.
- Stop and report what was done if a rate-limit window passes 70 percent.

## Method

- Start from the forked fan bench's harness under its `data/harness/`.
- Give the handler the same context the Claude fan's round 1 on `P4` had: its prompt with the PATTERN overview inlined as production renders it, and the snapshot's `CLAUDE.md` in the snapshot's root as `AGENTS.md`.
  This matches the sibling task `codex-astra-context-parity-bench`, so the two runs differ only in the prompt addition below; use the same context files it uses.
- Add one block to the handler's prompt and to each fork's, written once and kept in `data/`, that tells the reviewer to:
  - check every claim a card makes about existing code by reading that code and its callers, and the code paths the change reaches beyond the cards;
  - read the `pattern/` background file of every PATTERN entry the plan touches, and check that each new command, package or file the plan adds appears where that entry requires;
  - read the tests the plan adds or changes for waits, sleeps and timing that would slow or flake the suite.
  The block names kinds of reading and never the defects the reference holds.
- Confirm from the rollout that the forks started, ran concurrently and inherited the handler's context, and that `AGENTS.md` was loaded; if either fails, stop and report.
- Record the wall time, the handler's and each fork's fresh input and output tokens, and the windows.

## Grading

Grade the handler's findings, and any fork finding the handler dropped, against the fan reference of `P4` as the earlier benches did: `shared`, `unique-real` or `false`, in `data/match.tsv`.
Report beside the forked fan bench's row and the Claude fan row, and say for each of the Claude fan's three `BLOCKING` findings whether this run found it.

## Output

`docs/research/codex-astra-tuned-prompt-bench-<date>/report.md` with its `data/`, landed as the run's commit.
