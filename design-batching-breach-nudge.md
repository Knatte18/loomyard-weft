# A hook nudges an agent back to batching the moment it breaks the rule

`batching-breach-nudge` · Task · enhancement, claude, shuttle · done

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

## Design (operator, 2026-11-02)

A lym PostToolUse hook reads the transcript tail and, only on a breach, returns one line of additionalContext.
The line names the breach and quotes the rule.
No denies, because a deny costs the turn it is meant to save.

The hook reads the current call's own arguments (`file_path`, `offset`, `limit`) from the hook input, and the calls before it from the transcript at the path the input names, grouped per assistant message.
Parallel calls of one message count as one message, and the hook fires once per message, never once per call.

A breach is one of three:

- Single-lookup chain: a run of consecutive messages that each hold exactly one Read, Grep or Glob call and nothing else.
- Read slice: a Read with `offset` or `limit` on a file under a line threshold, or on a file already read and unchanged since.
- Edit-only chain: a run of consecutive messages that hold only Edit or Write calls.

The hook fires only when the session's context is above a context threshold, read from the transcript's usage figures.

Every number and switch is config, in a config file under the hub's config layers so one machine or hub can change it without a build, with the current values as the template defaults:

- the hook on or off, and each of the three breaches on or off;
- the context threshold, 150,000 tokens;
- the single-lookup run length that fires, 3 messages;
- the edit-only run length that fires, 2 messages;
- the line count under which a sliced Read of a file is a breach, 2,000;
- the number of times one session may be nudged, so a session that ignores the line is not nagged on every turn.

The writer picks the config module from the repository's config conventions and states its keys in the module's doc.

The rule wording changes alongside:

- override the Read tool's partial-read advice by name and forbid grep-then-slice;
- bind "wait for a result" to "then send every call it unlocks and every call you know";
- add "list every question, then send every grep and read for all of them in one message";
- widen the plan-edits rule to every round of edits, review fixes included;
- require a re-read in one message after a command rewrites files.

The hook is a heuristic: some lookups must run in sequence, so it will at times fire on a right call, which is why it only adds a line and the thresholds are config.
`batching-hook-effect` measures its effect once it has landed.
