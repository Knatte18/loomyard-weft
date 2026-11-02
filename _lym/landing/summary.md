# Nudge a session back to batching when its calls break the rule

Agents above about 150k context drift back into grep-then-slice chains and one-fix-per-turn edits, because both batching rules reach a session once, in the first user message.
In a 28-session sample, chained single lookups and edits spread over consecutive turns made up about a third of the context tokens.
This change adds an after-tool-call hook that adds one line of context on a breach, and sharpens the shipped rule wording.

## What changes

- **Hook.** A claude run's settings gain a PostToolUse hook, matched to Read, Grep, Glob, Edit and Write, that runs the hidden `lym shuttle hook-batching claude <state file>` verb.
  The verb needs no hub and is skipped by the shuttle group's geometry resolution.
  The command ends in `; true`, so a missing or failing `lym` never shows the session a hook error.
- **Judgment.** The claude engine reads a bounded tail of the transcript at the hook input's `transcript_path`, groups calls per assistant message, and answers with one `additionalContext` line that names the breach and quotes the rule.
  It never denies.
  Parallel calls of one message count as one message, and the hook nudges at most once per message.
  It stays silent below the context threshold, read from the transcript's usage figures.
- **Three breaches.**
  - Single-lookup chain: consecutive messages that each hold exactly one Read, Grep or Glob call and nothing else.
  - Read slice: a Read with `offset` or `limit` on a file under the line threshold, or on a file already read and unchanged since.
  - Edit-only chain: consecutive messages that hold only Edit or Write calls.
- **Config.** A `batching_nudge` block in `claude.yaml`, with defaults in `template.yaml`:

  | Key | Default |
  | --- | --- |
  | `enabled`, `lookup_chain`, `read_slice`, `edit_chain` | `true` |
  | `context_tokens` | 150000 |
  | `lookup_run` | 3 |
  | `edit_run` | 2 |
  | `slice_lines` | 2000 |
  | `max_nudges` | 5 |

  Values are validated when a run starts and on `lym config claude`.
  Prepare copies them into a per-run `batching-nudge.json` beside the events file, together with the nudge count, so a running session keeps the values it started with.
  With `enabled: false` the settings carry no hook and no state file is written.
- **Rule wording.** The shipped `lym-batching` and `lym-plan-edits-before-gate` entries (`internal/heddleengine/shipped.json`) now:
  - override the Read tool's partial-read advice by name and forbid grep-then-slice;
  - bind "wait for a result" to "then send every call it unlocks and every call you know";
  - add "list every question, then send every grep and read for all of them in one message";
  - widen the plan-edits rule to every round of edits, review fixes included;
  - require a re-read in one message after a command rewrites files.

## Design decisions

- The shuttle engine layer gains an optional `BatchingNudger` capability and the `BatchingHookVerb` constant, so an engine without the capability installs no hook and the verb stays provider-neutral.
- The hook is a heuristic, since some lookups must run in sequence and it will at times fire on a right call.
  That is why it only adds a line and why every number and switch is config.
- `max_nudges` caps the nagging of a session that ignores the line.

## To check by hand

- `hook-batching` has not been run against a live session; check that a session above the context threshold gets the line on a chain and none on a single right call.
- The nudge's transcript fixtures do not include one assistant message split across several transcript lines; check that a split message is still counted as one message.
- `docs/overview.md` and the package docs of `claudeengine` and `shuttleengines` were updated; skim them for accuracy.

`batching-hook-effect` measures the effect once this has landed.
