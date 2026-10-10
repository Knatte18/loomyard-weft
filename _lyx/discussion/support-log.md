# Support log: cli-surface

## Interview

The session ran autonomously; no operator answered.
Each turn below is a question the producer put to itself after reading the board entry and the code, with the pick it made.

1. Is this the right thing to build, rather than an MCP server?
   Yes: the board entry already weighs the MCP route and rejects it because an MCP server loads into every spawned session; the `lyx-mcp` note stays a note.
   The three parts share one root cause, an uneven CLI surface the orch rediscovers through `--help`, so they belong in one task.
2. Where does `lyx help index` live?
   Cobra's built-in `help` command is replaceable through `SetHelpCommand`; a `help` module keeps `lyx help [command]` and adds `index`.
   The alternative, a root verb `lyx index`, was rejected because the operator named `lyx help index`.
3. Where does the index renderer live?
   `internal/clihelp`: both `cmd/lyx` and every `*cli` package import it, and `orchcli` needs the same renderer for the role file.
4. How does the index reach the orch?
   The board entry says "same mechanism as the skill loading".
   Reading `watcher.go` showed the reload after a compaction skips the skills step, since the watcher assumes a compaction keeps skills, while the operator wants the index after every cycle because a compaction drops what was loaded.
   The role file is re-read on every pointer, so a marker in the role stencil reaches every path with no new typing step.
   Picked the role file; recorded as a departure from the entry's wording.
5. Does a group carry an `audience`?
   No: a group's audience is the union of its leaves; the test refuses an annotation on a group so the rule has one reading.
   A shared verb body mounted under several groups takes its value per mount.
6. How are the board payload keys shown?
   The operator's final wording: the payload form is the same for every board command, stated once, and each `Use` names its keys.
   `upsert` spells the full task key list; `upsert-batch` and `merge` reference `<task>`, defined once in the group's `Long` and on the index's `board` line.
7. What is enforceable on `Long`?
   A byte cap; content is review.
   Set at 2,000 bytes, between the measured median (about 740 bytes) and the ten largest pages (2.5 to 5.4 KB).
8. What does a carried comment look like?
   `## Issue comment, <date>`, the form the operator used when completing #479 and #480 by hand; one level deeper under a fold's `## From issue #N`.
   Author not recorded.
9. How is lyx's own pointer comment kept out?
   Its one form is a constant the writer and the filter share.
10. Does `intake list` need an extra call per issue for the count?
    No: GitHub's issue object carries a comment count.

## Rejected alternatives

- **An MCP server over lyx.** Loads into every spawned session, so every role pays for tools it must not use; the CLI already exists and is the same for an agent and a person.
- **`lyx index` as a root verb.** The operator named `lyx help index`; a root verb would also sit beside the modules in `lyx --help` as if it were one.
- **Typing the index into the orch as a skill load (`ly:cli` in `orchSkills`).** Misses the reload after a compaction, where the watcher skips skills; shuttle's `Send` refuses multi-line text, so the raw index cannot be typed either.
- **A separate `.lyx/orch/command-index.md` file named by the role file.** One more read per launch for no gain over a marker in the role file.
- **Audience inherited from the group.** Fewer annotations, but a leaf's audience then depends on where it is mounted and the test has two readings; explicit per leaf is one rule.
- **Hiding `role` and `internal` commands from `--help`.** The operator decided on 2026-10-09 that the annotation decides what the orch is handed, never what help shows.
- **A `<addr>` shorthand for `{slug|id}`.** Short enough to spell literally; a shorthand would need a legend for three characters saved.
- **Flag descriptions in the index.** Measured at another 14 KB; the orch reads `--help` for a flag's meaning.
- **A full-path index line (`lyx batten goto …`).** 11 to 12 percent larger than the nested form for the same content.
- **Recording the comment author in the heading.** Not in the operator's hand-made form; the issue link above the body gives the context.
- **Dropping every lyx-authored comment by author.** The author of a `gh`-token comment is the operator's account, so authorship does not identify lyx; the pointer's form does.

## Review rounds

_No rounds yet._

## Question ledger

| # | Question | Resolution |
|---|---|---|
| Q1 | Build the CLI index, not an MCP server? | Resolved from the board entry: CLI index. |
| Q2 | Where does `lyx help index` mount? | Auto-pick A8: a `help` module via `SetHelpCommand`, keeping `lyx help [command]`. |
| Q3 | How does the orch get the index? | Auto-pick A1: a marker in the role file, filled at every render; departs from the entry's "same mechanism as skill loading", reasons in D4. |
| Q4 | Do groups carry `audience`? | Auto-pick A2: no; union of leaves. |
| Q5 | Borderline diagnosis verbs? | Auto-pick A3: `operator`. |
| Q6 | How much of the task key list does `upsert`'s `Use` spell? | Auto-pick A4: all of it; `upsert-batch` and `merge` reference `<task>`. |
| Q7 | `Long` cap value? | Auto-pick A5: 2,000 bytes, the test's constant. |
| Q8 | Comment heading form? | Auto-pick A6: `## Issue comment, <YYYY-MM-DD>`, date only. |
| Q9 | Operator skill name? | Auto-pick A7: `ly:cli`. |
| Q10 | Registration oracle and `SetHelpCommand`? | Open for the plan: mount through `AddCommand` too, or extend the oracle; listed under Constraints. |
| Q11 | Does `lyx --json` list the new `help` module? | Open for the plan; listed under Open risks. |
| Q12 | Optionality of `merge`'s keys in `Use`? | Open for the plan: read from the parser; the coupling test pins it. |
| Q13 | Environment | `lyx board get` and `lyx board list` ran clean; nothing outside the two output files was touched. |
