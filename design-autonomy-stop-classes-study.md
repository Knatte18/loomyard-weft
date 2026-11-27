# Study every run stop of the last two weeks and propose who settles each class under autonomy

`autonomy-stop-classes-study` · Task · feature, dobby, batten, orch

## Aim

Give the design of the entry "Autonomy settles known run stops itself, so lym runs overnight and only severe halts reach the operator on Slack" its facts:
which stops runs hit, how often, and who settled them how.

## Sources

This hub (loomyard) on this machine only, from 2026-11-13 to 2026-11-27:

- the runs' drive reports, escalation briefs and run records;
- dobby's notices and run-watch mail, and creel mail to and from the orch;
- the orch's session transcripts, for what the orch and the operator did about each stop.

No run, test, trial or probe is run; the study reads records only.

## Report

`docs/research/autonomy/stop-classes-2026-11-27.md`, holding:

1. A table of stop classes: the class, its count, an example run, who settled it (lym, the orch, the operator) and how, and the time each stop held its run.
2. For each class, whether it is mechanical (a bounded retry or a fixed answer settles it) or needs judgment, and what judgment.
3. A proposed rule table with three outcomes per class: lym settles it itself, the orch settles it under autonomy, or it is a severe halt that waits for the operator and reaches them on Slack.
   Split the orch column by two autonomy levels: today's, and a stricter overnight level in which the orch never asks the operator.
4. Where in lym's code each lym-settled class would be handled (dobby, batten, the driver), by package.
5. The notices that fired for an agent that was waiting correctly, and what each should name instead.
