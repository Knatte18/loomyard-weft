# TwillDiscusser: a Discussion writer that forks lens reviewers before it writes

`twill-discusser` · Task · enhancement, loom, stencil · running · Publish

## Seen (2026-10-11)

The first Discussion bench (`docs/research/discussion-bench.md`, 7d439cf71) found that no seats chair with an advisor wrote a better draft than the same chair alone, and that an Opus or high-effort Sonnet advisor roughly doubled the cost.
Each seats advisor is a standalone session at full price that re-reads the code itself; the operator names this as part of the problem.
Each configuration ran once on one task, so the noise is unmeasured.

## Idea (operator, 2026-10-11)

`TwillDiscusser`: a new Discussion-Write producer, chosen with `discussion_producer: twill` beside `single` and `seats`, that brings Burler's lens fan into the writer's own session.
To shed it is a single-LLM producer; the forks live inside the writer's Claude Code session, so the new parts are its stencil, the lens texts and a wait command, not the shed engine.

1. The writer, on a strong model such as `opus[medium]`, explores the task and code in full on its own.
2. Ready to write, it does not write yet: it forks itself once per lens.
3. Each fork is told to wait until the decision record exists, and does no lens work before then.
4. The writer, which is not a fork, writes the whole decision record and support log.
5. Each fork then reviews the written record through its lens and returns its findings to the writer.
6. The writer takes in the findings and finalises the record.

Gains:

- Every fork runs on the writer's strong model.
- Every fork inherits the writer's exploration, so it reads that prefix from cache rather than re-reading the code.
- The forks did not write the record, so they read it as outsiders rather than defending decisions they made; forks spawned after the draft would carry its writing in their context and tend to polish it rather than question it.
- No seats engine or SendMessage rounds.

Lenses: each fork gets a focus, as Burler's lens forks do (for example correctness, error handling, test gaps, scope, consistency with the board); Burler's fan lens texts are the starting point.

## Cache while the forks wait

Writing the record often takes longer than the 5-minute prompt-cache lifetime, so a fork that waits idle would re-read its whole context uncached when it wakes, losing the cache gain.
Nothing outside can inject a ping into a fork, so each fork keeps its own cache warm the way the Burler fixer does with `lyx burler await` (`internal/burlercli/await.go:25`):

- the fork runs a foreground wait for the record file, capped at about 4 minutes;
- if the cap is reached first, the fork takes one short turn that only re-runs the wait, which reads its context from cache and resets the 5-minute clock;
- when the record appears, the wait returns at once and the lens review starts.

This needs a general wait command (wait for a file, with a cap), since `lyx burler await` is built for the fixer; the same command serves the writer waiting on parent review (`parent-review-cold-cache`).

## Scope of this task

Build the `twill` producer, its stencil and lens texts, the general capped wait command, the config wiring and the docs.
The bench below runs after landing, from the prime, as its own job.

## Later: Plan

The same pattern fits the Plan writer, whose review findings are often enumerable (a test missing from a card's `Edit:` list, fixtures not enumerated) and whose exploration is larger, so forks save more; it comes after `TwillDiscusser` has been benched (operator, 2026-10-11).
Keep the fork-wait-review stencil part and the wait command reusable for that.

## Bench

Compare, three runs each on the same reference task, so the spread is measured too:

- `opus[medium]` with `TwillDiscusser`;
- `opus[medium]` alone;
- `opus[high]` alone, the first bench's best draft.

Measure wall time, price per configuration, the draft against the reference final discussion judged on the merits, and where possible how many Discussion-Review rounds each draft needs.
The open question whether Burler's fan findings are worth their cost applies here too.
