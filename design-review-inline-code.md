# Paste cited code into a review rounds context

`review-inline-code` · Note · enhancement, burler, stencil

## Seen

Note `kickstart-per-card` rejects pasting span code into a Webster fork's pack: the code goes stale as earlier cards edit the files, and Claude Code refuses an Edit on a file the agent has not read itself, so an implementer must read every file it edits anyway.
Neither reason holds for a reviewer: it edits nothing, and it reviews a fixed commit, so code pasted at the round's start cannot drift during the round.

## Operator view, not a decision (2026-10-15)

For reviewers it may be different: they could get the code the target cites pasted in, instead of reading it themselves.

## Operator decision this note must answer (2026-10-16)

From the stencil intent audit of 2026-10-15 (now in note `webster-fork-stencil-fixes`): code text is never pasted into a prompt without a good reason, and the reason is written down.
Before this note is built, it states its good reason for pasting code into a review round against that decision.

## Earlier data

Note `fan-master-preread` records one hand-built try on solo `opus[medium]` Discussion review (2026-10-10, one run on the `webster-replan-state` fixture): a body dump of the 35 cited symbols, about 20k tokens in a file read first, cost about the same as a kick-start pack of spans and missed the BLOCKING finding the span pack found.
One run per arm is too few to decide anything.

## Idea

Give a review round, Webster-Review or Merge-Review first, since their target is a diff at a fixed commit, the bodies of the symbols the target changes or cites, rendered by lyx at round start, and measure finding quality and cost against today's reviewer.

## Problem raised by the operator (2026-10-15)

A reviewer often finds it must explore more code than it was given.
Code pasted into its context does not count as read the way its own Read calls do, so it cannot tell what it already holds: it may re-read pasted code or miss that a span is already there.

Possible answer (the orch's idea, not discussed): label each pasted block with its file and line range, and open the paste with a list of every file and range it holds, so the reviewer checks that list before a read.
The `fan-master-preread` lab used the same kind of reading manifest for forks; whether reviewers follow it is untested.

## A glyph pack for reviewers (from note `glyph-use-audit`, 2026-10-15)

The Burler side of the kick-start pack, issue #228: a pack of the reviewed glyphs injected once before the reviewer's explore phase, so every lens fork inherits it (the operator's architecture correction on #228).
Measured basis from before: quarry's `results/2026-09-06-kickstart` (`bench/loomyard-eval/ladder/results/2026-09-06-kickstart/conclusion.md`, commit `d2e492d`).
The Webster pack audits (`docs/research/glyph-audit-2026-10-12`, `-10-13`, `-10-15`) found too little gain to build it yet: forks read about 20 % less but explore as much.
Take it up only if the per-card pack of task `kickstart-per-card` shows a clear gain when measured with `tools/forkaudit`.

### The pack is the read set (was note pack-is-the-read-set)

Seen in fan-lab 2026-10-10: a solo `opus[medium]` reviewer given a kick-start pack ("read all listed spans first") read the pack and then explored about 20 more tool calls, so the pack came on top of exploration instead of replacing it.
Neither the prompt nor the ladder card said to stay with the pack.

The reviewer prompt would say the pack is the read set:
- the reviewer reads the pack's spans and treats them as read;
- any read beyond them needs a named gap (a caller or spawned process one hop out), within a small tool-call budget;
- the reviewer ends with one line per extra read saying why the pack lacked it (the fork rule from `fan-master-preread` variant D).
