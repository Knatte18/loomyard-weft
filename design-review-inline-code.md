# Paste cited code into a review rounds context

`review-inline-code` · Note · enhancement, burler, stencil

## Seen

Note `kickstart-per-card` rejects pasting span code into a Webster fork's pack: the code goes stale as earlier cards edit the files, and Claude Code refuses an Edit on a file the agent has not read itself, so an implementer must read every file it edits anyway.
Neither reason holds for a reviewer: it edits nothing, and it reviews a fixed commit, so code pasted at the round's start cannot drift during the round.

## Operator view, not a decision (2026-10-15)

For reviewers it may be different: they could get the code the target cites pasted in, instead of reading it themselves.

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
