## Goal

```markdown
Fix the faults the first Codex engine trial found (`docs/research/codex-engine-trial-home-sandbox-2026-11-03.md`), before the next Codex trials run.

## 1. `lym shuttle run --model` takes a model spec

`lym shuttle run --model luna[medium]` passed the string to shuttle.yaml's engine (claude) as a provider model id; Claude stopped with `model_not_found`, the run sat to its timeout and left its strand alive.
`lym shuttle open --model` already takes a model spec and picks the engine from it.

Wanted (operator, 2026-11-04):

- `run --model` takes the same model spec as `open`: `<alias>[<effort>]` or `<engine>:<model-id>[<effort>]`, such as `sonnet[medium]` or `luna[high]`.
- The model resolver picks the engine from the alias: `sonnet` runs on claude, `luna` on codex, as the model registry says.
- A model the engine rejects fails the run at once, never a wait to the timeout.

## 2. `lym shuttle open` and the codex engine

`lym shuttle open --role x --model 'luna[medium]'` resolves codex and is refused: `the engine "codex" cannot open a session with no prompt`, while `--list-models` lists the codex aliases.

Wanted: `open` starts an interactive Codex session with no prompt; where Codex cannot, `open` refuses a codex spec before it creates anything, and `--list-models` and the help say which engines `open` can start.

## 3. A test writes the real `~/.lym/codex`

`TestPrepare_AppendsStandingPassages` in `internal/shuttleengine/codexengine/prepare_test.go` sets `engine.dirs.Board = t.TempDir()` after `newEngine(anchor)`, so the fixture's codex.yaml is no longer found, the config resolves the template's `~/.lym/codex` and `~/.codex`, and each gate run adds `/tmp` roots to the real `lym-trust.json` and `config.toml`.

Wanted: the test keeps the fixture's home; no test reaches the operator's home.
Remove the `/tmp` test roots the test left in `~/.lym/codex` is not part of the change: the next real launch drops roots whose folder is gone.

## 4. The committed-`.codex` check

- The refusal and the Warn fire on a `.codex/rules` folder or `.codex/config.toml` that git does not track; the package doc says "committed". Make the code and the doc agree, choosing whichever the safety argument wants.
- The refusal's way forward ends in "clear the round's assist model key", which fits a review round and not a plain `lym shuttle run`; word it for the launch at hand.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
