# Give lym shuttle run a per-run advisor option

`lym shuttle run --advisor <model>` gives one run Claude Code's server-side advisor tool with the named model.
Until now no route set the advisor for a single run: claudeengine's `--settings` document carried hooks and scope only,
and heddle's `claude-settings` target and the `binary` wrapper reach every session of a hub.
The advisor bench (`advisor-model-bench`) needs this route.

## What changes

- `shuttleengine.Spec` gains `Advisor`, a provider-vocabulary model name that Spec validation passes through unchecked.
- claudeengine writes a non-empty `Advisor` into the run's `settings.json` as `advisorModel`.
  An empty one leaves the key out, so a launch without `--advisor` writes the same document as before.
  The resume line names the same settings file, so a revived session keeps its advisor.
- codexengine refuses a spec that names an advisor before it writes anything, naming the way forward, since Codex has no advisor tool.
- `lym shuttle run` takes `--advisor`.
  Its flag usage states the value form, the capability rule and the Claude Code version a launch confirmed it on,
  and its `Long` help names the flag in one sentence, under the help's byte cap.
  No lym config key is added.
- `buildSettings` takes the advisor as one more argument, so its existing test call sites pass `""`.

## Value form and effect, confirmed

Claude Code 2.1.322 documents its hidden `--advisor` flag as taking an alias or a full model id, and resolves `advisorModel` the same way.
A `sonnet[low]` run through `lym shuttle run --advisor opus` in a lym tmux session showed `advisorModel` resolved to `claude-opus-5-5`,
and the session called the `advisor` server tool and got its answer.
Claude Code also requires the advisor to be at least as capable as the session's model, which the flag usage states.
How advisor calls show in the usage records is left to the bench.

## Design decisions

- The advisor rides the `--settings` document rather than the `--advisor` flag,
  since claudeengine already writes that document for every launch and the resume line reuses it.
- An engine without an advisor refuses the field, the way codexengine already refuses a permission mode it cannot realize;
  no new `Requirement` constant is added, since one engine refusing the field covers the case.

## Tests

- `TestPrepare_AdvisorReachesSettings` pins `advisorModel` in the written `settings.json`, and its absence when no advisor is named.
- `TestPrepare_RefusesBeforeWriting` in codexengine gains an advisor row.
- `TestRunCmd_ModelSpec` gains a row showing `--advisor` reaching the engine's spec unchanged.
