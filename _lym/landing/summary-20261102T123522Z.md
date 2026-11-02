# A hook nudges a session back to batching; sharper batching rules

Above about 150k tokens of context, agents drift into grep-then-slice chains and one-fix-per-turn edits, even though the batching rules reach every session through `CLAUDE.local.md`.
This change adds a hook that adds one line to the session's context the moment a breach shows. It never denies a call.
It also rewords the two shipped heddle rules the hook quotes.

## What changes

**The hook.** `claude.yaml` gets a `batching_nudge` block (`internal/shuttleengine/claudeengine/template.yaml`) with these keys:

- `enabled`
- the breach switches `lookup_chain`, `read_slice` and `edit_chain`
- the thresholds `context_tokens` (150000), `lookup_run` (3), `edit_run` (2), `slice_lines` (2000) and `max_nudges` (5)

The keys are in the `CategoryNone` category, so the hub base, a worktree file or a machine's local override can change them without a build.
When the hook is on, `Prepare` copies the block into `batching-nudge.json` in the run directory.
The run's `settings.json` then gets a PostToolUse entry that matches `Read|Grep|Glob|Edit|Write` and runs `lym shuttle hook-batching claude <state file>; true`.
Other tool calls start no process.

The hidden verb (`internal/shuttlecli/hookbatching.go`) reaches the engine through a new optional capability, `shuttleengine.BatchingNudger`, which `shuttleengines.BatchingNudger` looks up by engine name.
The verb skips shuttle's hub resolution and always exits 0.

The judgment lives in `claudeengine/batchingnudge.go`.
It reads the call's own input (`file_path`, `offset`, `limit`) and a bounded tail of the transcript the input names.
It groups the main-chain assistant lines by message id, so parallel calls in one message count as one message.
It says nothing in these cases:
- the call comes from a subagent
- the latest usage reading is below `context_tokens`
- the run has used up `max_nudges`
- the hook already nudged this message

It fires on the first of these breaches that applies:
- **Single-lookup chain:** at least `lookup_run` messages in a row, the current one included, each with exactly one Read, Grep or Glob call.
- **Read slice:** a Read with a `limit`, or with an `offset` past line 1, of a file under `slice_lines` lines, or of a file an earlier message read and that has not been modified since.
- **Edit-only chain:** at least `edit_run` messages in a row with only Edit or Write calls.

The judgment and the count update run under the state file's lock (`state.UpdateJSON`), so the parallel calls of one message get at most one nudge.
`ValidateBatchingNudge` refuses a negative `context_tokens`, or any other threshold below 1, at a run start and on `lym config claude`.

**The rules** (`internal/heddleengine/shipped.json`).
`lym-batching` now:
- overrides the Read tool's own advice to read only the part you need, by name, and forbids grep-then-slice
- follows "wait for a result only to choose the next call" with "then send every call it unlocks and every call you know"
- asks for every question's greps and reads in one message
- requires re-reading, in one message, the files a command rewrote

`lym-plan-edits-before-gate` now covers every round of edits: a plan, a review's findings, a fixer's report or a failed gate as well as a card or task.

## Design decisions

- **Config module:** the hook is Claude-specific (tool names, transcript format), so its keys live in `claude.yaml` with the engine. This follows the provider seam.
- **Settings at the start of a run:** the state file holds the settings the run started with. The hook process therefore loads no config and needs no hub, and a running session keeps its values, the same way it keeps its `refuse_raw` list.
- **The current message as the transcript shows it:** the current message counts toward a chain as far as the transcript shows it. If it already shows a second call or a non-lookup call, the chain is broken and the hook stays silent.
- **One-line ordering:** when more than one breach applies, the line names only the first, in the order the design lists them.
- **Change test for a re-read:** "unchanged since" compares the file's modification time with the earlier reading message's timestamp. The session's own Edit therefore counts as a change, which errs toward silence.

## Bounds

- The hook is a heuristic: some lookups must run in sequence, so it will sometimes fire on a correct call. That is why it only adds a line and why every threshold is config.
- The hook for an early call can run before the message's later calls are in the transcript. In that case a batched message that follows a chain can read as one more single lookup.
- An earlier read outside the transcript-tail window, `batchingTailBytes`, is not seen.
- A call whose tool fails fires no PostToolUse hook, so it is not judged.
- `batching-hook-effect` (R3) measures the effect once this lands.

## Tests

- `TestBatchingNudge` covers each breach firing and every case that holds a nudge back, plus the count the state file keeps.
- `TestPrepare_BatchingNudgeHook` covers the hook entry and the seeded state file, the off switch, and the refusal before any artifact is written.
- The template's values, the `CheckLoad` refusal, the registry lookup and the pinned `lym-batching` text are rows in their existing tests.
