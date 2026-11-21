# Unjudged red-first refusal burns the verify attempts before the parent answers

`red-first-unjudged-burns-tries` · Note · bug, gate, loom

## Seen

2026-11-21, in `gate-dotnet-fixes` and `red-first-overlay-non-go-files`: the darn writer asked the parent for a "No test seam" decision, then re-ran the verify gate at once.
The unjudged red-first refusal repeated, the run spent its three verify attempts within about two minutes and stopped with a question,
while the parent's `lym loom decision add` was still committing. The parent then had to resume the run by hand.

## Idea

An unjudged red-first refusal (exit `RedFirstUnjudgedExit`) does not spend a verify attempt; the writer waits for the parent's reply,
or the gate waits for the decision record to change before it re-runs.
