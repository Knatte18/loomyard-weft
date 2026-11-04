# lym shuttle run --anchor hidden ends died on both engines: the provider never becomes ready

`shuttle-hidden-anchor-dies` · Note · bug, shuttle, reed

Found in the codex-trial-hooks-guards-skills trial, 2026-11-04.

## What happens

`lym shuttle run --anchor hidden --model 'luna[medium]' --role x --prompt ... --output-file ...` from a worktree whose reed session is live ends with outcome died after two start attempts about 10 s apart.
The log says `shuttle: provider never became ready; strand torn down` and the error says `no pane capture was saved; the strand was removed`.
The same run with the default own-window anchor starts and finishes.

## Reproduced

Three hidden runs, three deaths: `luna[medium]` twice and `sonnet[low]` once. So the fault is not in codexengine.
The sonnet death carried a sessionId, so Claude started a session before the strand was torn down.

## Expected

A hidden strand reaches the provider as an own-window one does, or the refusal names why a hidden anchor cannot run.

## Not known

Which of reed (hidden placement) or shuttle (the readiness read of a hidden pane) is at fault; both labels are set until one is cleared.