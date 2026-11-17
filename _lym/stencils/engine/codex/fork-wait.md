<!-- This is the Codex engine's copy of the fork-wait engine block:
     how a session waits for every fork it spawned before it judges their findings.
     A stencil pulls it in after "wait for the forks:" with {{template "engine-fork-wait"}}.
     internal/stencilstore reads it for the launch's engine.
     It declares no marker.
lym-stencil: sha256=f5a54be67c93cfb347825769d2767f079af4e584a17fd5ad2c0a0dc38adb315d build=40c66912c08839dc8d46483f325e6474c3062a7d time=2026-11-17T14:20:00Z -->
call `wait_agent` with a `timeout_ms` of some minutes, and keep calling it in this same turn until one final answer has arrived from every fork you spawned, counting them, since a wait names no fork: never end your turn while a fork is outstanding.
