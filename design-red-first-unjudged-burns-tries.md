# Unjudged red-first refusal burns the verify attempts before the parent answers

`red-first-unjudged-burns-tries` · Note · bug, gate, loom

## Seen

2026-11-21, in `gate-dotnet-fixes` and `red-first-overlay-non-go-files`: the darn writer asked the parent for a "No test seam" decision, then re-ran the verify gate at once.
The unjudged red-first refusal repeated, the run spent its three verify attempts within about two minutes and stopped with a question,
while the parent's `lym loom decision add` was still committing. The parent then had to resume the run by hand.

## Idea

An unjudged red-first refusal (exit `RedFirstUnjudgedExit`) does not spend a verify attempt; the writer waits for the parent's reply,
or the gate waits for the decision record to change before it re-runs.


## From issue #173

Imported from [issue #173](https://github.com/Knatte18/loomyard/issues/173).

**What happened.**
Darn-Write halted blocked (anomaly escalation-to-human) after the chair's `lym gate red-first --gate darn-write` failed 3 attempts in a row with the same error.
The error: the diff changes a contracts spec markdown file, which the overlay cannot put back, so the base run cannot judge the tests.
The refusal's way forward is for the run's parent to record a `lym loom decision add` entry titled "No test seam".

**Why abnormal.**
The agent receiving the findings cannot resolve this itself; it needs a parent decision.
The gate re-prompted the agent three times, so the whole verify budget went on an identical, deterministic refusal, and the run ended blocked.
A diff that pairs a Go test with a doc update is ordinary; the gate treats the doc as making the whole check unjudgeable.

**What to do.**
Make the red-first check ignore non-Go doc files that cannot affect test outcomes, or restore only the Go files on the base.
When it must refuse, escalate to the parent on the first attempt instead of re-prompting the agent through the full budget with an unactionable finding.

Provenance: task raddle-slug-length-check; note loom-halt.md.
