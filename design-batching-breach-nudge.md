# A hook nudges an agent back to batching the moment it breaks the rule

`batching-breach-nudge` · Note · enhancement, claude, shuttle

## Seen (orch, 2026-11-01)

Both batching rules reach every session, but only once, in CLAUDE.local.md inside the first user message.
Above about 150k context agents drift back to grep-then-slice chains and one-fix-per-turn edits.
In 28 sessions, single-lookup turns that follow another single lookup above 150k come to 128 turns and 34.7M context tokens.
Edits spread over consecutive turns come to 39 turns and 15M.
Together that is about a third of the sample's 147M.
psb-rv-plan:plan alone has 31 chained grep turns at 390k–530k and 15 one-file-per-turn fix turns at 630k–668k.
The Read tool's own description ("only read that part") is fresh in every request and argues against "read a file whole".
"wait for a result only to choose the next call" licenses the chains.
The plan-edits rule's "change/card/gate" wording does not reach planners or review fixes.
The darn writer's read, write and read on test files came from Edit's stale-read guard after its own rewrite program; it is no habit.

## Idea

Add a lym PostToolUse hook that reads the transcript tail and, only on a breach, returns one line of additionalContext.
A breach is the third single-lookup message in a row, a Read slice of a file under 2,000 lines or of an already-read unchanged file, or an edit-only message after an edit-only message.
The line names the breach and quotes the rule.
Change the wording alongside:
- override the Read tool's partial-read advice by name and forbid grep-then-slice;
- bind "wait for a result" to "then send every call it unlocks and every call you know";
- add "list every question, then send every grep and read for all of them in one message";
- widen the plan-edits rule to every round of edits, review fixes included;
- require a re-read in one message after a command rewrites files.

No denies, because a deny costs the turn it is meant to save.

## Two ways, to weigh when promoted (operator, 2026-11-01)

- Breach hook, as above: fires at every breach, about 6 lines a session in the sample.
  It reads the call's own arguments (`file_path`, `offset`, `limit`) from the hook input and the transcript tail for the calls before it.
  Its cost is the breach definition: some lookups must run in sequence, so it will at times fire on a right call.
- Threshold reminder: lym re-sends the batching rules once when a session's context passes 150k, and again at 300k.
  It reads no calls, so it is simpler and quieter, but it does not say which call broke the rule.
- The rule wording changes above go with either.
