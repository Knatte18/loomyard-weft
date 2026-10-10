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
   Set at 2,000 bytes, between the measured median (about 740 bytes) and the ten largest pages (2.5 to 5.4 KB), which make up the bulk of the 40k tokens a full `--help` dump costs.
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
- **A full-path index line (`lyx batten goto …`).** 11 to 12 percent larger than the nested form for the same content; the operator measured the nested form at about 3.5k tokens for the `operator` audience.
- **The watcher rendering the index from its own command tree.** The `lyx orch watch` process outlives a deploy, so its tree describes the old binary while the orch calls the new one.
- **The watcher running `lyx help index` through its own `PATH`.** The watcher is a detached daemon without reed's pane prelude, so its `PATH` can name a different `lyx` from the orch pane's, such as the installed one while the orch runs from `.dev-bin`.
- **Exempting `help` from the whole `cmd/lyx` tree walk.** Would leave `help` and `index` outside the audience and form tests; only the bare-group check conflicts with keeping the built-in's root help.
- **Keeping the built-in's `Find` fallback for `lyx help <group> <unknown>`.** Prints the group's help for a mistyped verb, where every other lyx group refuses an unknown subcommand.
- **The watcher running its own executable's path, recorded at its start.** Matches the pane only when one `lyx orch start` both launches the strand and spawns the watcher; a `start` that only spawns a watcher for a live strand, or a reed resume relaunching the pane from another `lyx`, leaves the two apart.
- **Leaving `startAutoReload`'s render error as a failed tick.** A missing binary would then spawn a failing `help index` every poll until the consecutive-error limit ends the watcher, where the other render sites go idle.
- **Restarting or refusing a watcher whose build stamp differs.** Interrupts a live orch's watcher on every landing's deploy, to fix one string the watcher can fetch from the current binary instead.
- **Equality of `Use` key names alone in the board coupling test.** Passes a wrong `?` or `|` marker; the round-1 review found `merge` already marking its required `upsert` optional.
- **Recording the comment author in the heading.** Not in the operator's hand-made form; the issue link above the body gives the context.
- **Dropping every lyx-authored comment by author.** The author of a `gh`-token comment is the operator's account, so authorship does not identify lyx; the pointer's form does.

## Review rounds

### Round 1

The review raised nine findings; all were fixed in `decision-record.md`.
The form test reads a `{…}` span as one token (D6); the coupling test compares required, optional and exclusive keys, and `merge`'s `upsert` lost its wrong `?` (D7); the watcher takes its index from the `lyx` on `PATH` (D4); runnable groups such as `lyx config` carry an audience and a leaf-style line (D2, D3); the index skips `completion` and unannotated commands (D2); D1 bounds the replacement help command and decides that `lyx --json` lists `help`; D11's pointer filter matches the whole trimmed body only.
Measurement narrative moved here, and the registration route is stated in Constraints only.

### Round 2

The review raised three findings; all were fixed in `decision-record.md`.
D1 states how `help` meets the `cmd/lyx` tree walk (exempt from the bare-group check alone, `operator` on `help` and `index`, `Use` `help [<command>...]`, with D6 gaining the `[<arg>...]` form) and refuses an unknown word at any depth of a help topic with `unknown subcommand`; D4 runs the watcher's index source from the absolute path the watcher was started from rather than its own `PATH`, with a bound for a missing path, and Constraints name `PATTERN-pane-binary-resolution`.

### Round 3

The review raised three findings; all were fixed in `decision-record.md`.
D4 has reed record each strand's prelude binary at every launch and relaunch, and the watcher's index source runs the orch strand's recorded path, since the watcher's own executable differs from the pane's on a watcher-only start and after a reed resume; `startAutoReload` goes idle on a render failure like the other two sites, the one change Scope admits to the reload steps; D9 exempts the root's module list, which the tree-walk test pins.

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
| Q11 | Does `lyx --json` list the new `help` module? | Decided in round 1 (D1): yes, with `index`; `completion` stays skipped. |
| Q12 | Optionality of `merge`'s keys in `Use`? | Read from the parser in round 1 (D7): `upsert` required, `remove_slugs` and `set_status` optional; the coupling test compares optionality. |
| Q13 | Environment | `lyx board get` and `lyx board list` ran clean; nothing outside the two output files was touched. |
| Q14 | Which binary does the watcher's index describe? | Auto-pick A9, revised in rounds 2 and 3: the absolute path reed records on the orch strand at each launch and relaunch, run at each render, not a `PATH` lookup or the watcher's own executable. |
| Q15 | How does `help` meet the `cmd/lyx` tree walk, and what does `lyx help <group> <unknown>` do? | Decided in round 2 (D1): exempt from the bare-group check alone; an unknown word at any depth is refused with `unknown subcommand`. |
