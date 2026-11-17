<!-- This is the Codex engine's copy of the fork-message engine block:
     how a session on Codex sends a message to a fork it spawned.
     A stencil pulls it into a sentence with {{template "engine-fork-message"}}.
     internal/stencilstore reads it for the launch's engine.
     It declares no marker.
lym-stencil: sha256=6d8e2bed47c9837b9d43c387116fca18118c97ff64097408650646f7a8123d9f build=40c66912c08839dc8d46483f325e6474c3062a7d time=2026-11-17T14:20:00Z -->
with `send_message`, addressed to the task name that `spawn_agent` returned for that fork
