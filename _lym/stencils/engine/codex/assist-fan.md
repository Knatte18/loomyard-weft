<!-- This is the Codex engine's copy of the assist-fan engine block: how an assistant reviewer on Codex runs its lens subagents inside its one turn.
     burler-assist-cluster-fan pulls it in with {{template "engine-assist-fan"}}; internal/stencilstore reads it for the assistant's engine. It declares no marker.
lym-stencil: sha256=794d1a6b7a83840f6aee09ed39f9b69f10aa40ad0594198ace2c63977d265bd7 build=935fb407d46b918a520439401a144eb28d3de274 time=2026-11-03T17:43:20Z -->
Spawn the subagents with `spawn_agent`, one per lens, each with `fork_turns: "all"` so it inherits everything you have read, all in one go, as soon as your exploration is done.
Then wait for every subagent with `wait` inside this same turn, and keep waiting until the last one has returned: never end your turn while a subagent runs, since a turn you end leaves your report unwritten.
