<!-- This is the Claude engine's copy of the fork-message engine block:
     how a session on Claude sends a message to a fork it spawned.
     A stencil pulls it into a sentence with {{template "engine-fork-message"}}.
     internal/stencilstore reads it for the launch's engine.
     It declares no marker.
lym-stencil: sha256=2f1f2c1ad20e2a02798322be12bfbb4e2286d3809f23adbcf3f2f8cdf485a031 build=40c66912c08839dc8d46483f325e6474c3062a7d time=2026-11-17T14:20:00Z -->
by `SendMessage`, addressed by the agent id its completion notification carried
