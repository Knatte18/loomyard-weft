# lym shuttle open refuses the codex engine because it has no prompt

`shuttle-open-refuses-codex` · Note · bug, shuttle, codex

Found in the codex-trial-home-sandbox trial.

`lym shuttle open --role x --model 'luna[medium]'` resolves the codex engine and is refused: `shuttle: open: the engine "codex" cannot open a session with no prompt`.

`open --help` lists the codex aliases (astra, luna, sol) in `--list-models` and names `<engine>:<model-id>` specs without saying a Codex model cannot be opened. Codex takes its first turn as the positional prompt, so open cannot start it.

Expected: `--list-models` and the help say which engines open can start, or open refuses a codex spec before it creates anything with a message naming the way forward (lym shuttle run with the engine set in shuttle.yaml).