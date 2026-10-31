# Landing commits end with a plain credit block, not co-author trailers

A landing commit no longer names an engine as a co-author, and no longer ends with a `Lym-Task: <slug>` line.
Instead the commit, the pull request's body and the pull request's merge message end with one credit block, written as plain lines rather than git trailers:

```
Created using
 - LoomYard https://github.com/Knatte18/loomyard
 - Claude <noreply@anthropic.com>
```

## Configuration

`landing.yaml` replaces `trailer_text` and `engine_trailers` with four cross-pair keys:

- `credit` (default `true`) turns the block on or off.
- `credit_heading` is the first line.
- `credit_loomyard` is the LoomYard item, listed whatever engines ran.
- `credit_engines` is an open map from engine name to that engine's item, listed only when the engine ran a session in the task, detected through the same engines-run read as before.

Go renders each item as a ` - <text>` line, engines in name order.
An empty text adds no line, and a block with no item left, or with `credit` off, is empty, so the message then ends at the body.
`credit_engines` keeps the old load check: a key naming no known engine fails the load and names the key.

`co_authored_by`, `trailer_text` and `engine_trailers` become retired keys.
A hub file still holding them loads, and `lym config reconcile` prunes them.
The old migration that moved `co_authored_by` into `engine_trailers.claude` is gone, since those values are the co-author lines this change removes.

## Pull request body

PR-Review now sets the pull request's body to the change description's trimmed body, a blank line and the credit block, so the commit and its pull request carry the same signature.
It reads the engines-run list each time it sets the body; a read failure is an error, as at Finalize.
Finalize's GitHub merge message is built by the same function.

## Design decisions

- The block is optional through an explicit `credit` bool rather than by emptying texts, since the LoomYard line is meant to be present whenever the block is.
- The template carries only the Claude engine line.
  `codex` is not yet an engine `modelspec` knows, so a `codex` entry would fail the engine-name check; the Codex engine's task adds `codex: "Codex <codex@openai.com>"` with the engine.
- The description gate's Co-Authored-By finding stays; only its detail text changes.

## Tests

- `TestConfig_CreditBlock` covers the block's order, deduplication, empty texts, an empty heading, no item line and `credit` off.
- `TestLoadConfig_TemplateDefaults` pins the new defaults and loads a file holding every retired key.
- `TestMigrateConfig` checks that the retired co-author and trailer keys are left for reconcile's prune with nothing carried into `credit_engines`.
- The Finalize message test asserts that neither `Lym-Task` nor `Co-Authored-By` appears.
- The PR-Review fixtures now run with the credit on, so every created and edited body carries the block.
- `TestFinalize_OverRealHub` checks the block in a real landing commit.
