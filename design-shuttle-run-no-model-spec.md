# lym shuttle run takes no model spec and has no engine choice

`shuttle-run-no-model-spec` · Note · bug, shuttle, codex

Found in the codex-trial-home-sandbox trial.

`lym shuttle run --model luna[medium]` passes the string to the engine in shuttle.yaml (claude by default) as a provider model id. The session started in Claude Code and stopped with `model_not_found`; the run then sat to its timeout as `timeout`, and left its strand alive until removed by hand.

`lym shuttle open --model` takes a model spec (alias or engine:model) and picks the engine from it, so the two verbs disagree on what `--model` means, and `run` has no flag to choose the engine.

Expected: `run --model` accepts the same spec as `open`, or its help says it is a provider id of shuttle.yaml's engine; and a model the engine rejects fails the run at once instead of timing out.