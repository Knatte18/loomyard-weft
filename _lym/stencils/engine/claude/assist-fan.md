<!-- This is the Claude engine's copy of the assist-fan engine block: how an assistant reviewer on Claude runs its lens subagents inside its one turn.
     burler-assist-cluster-fan pulls it in with {{template "engine-assist-fan"}}; internal/stencilstore reads it for the assistant's engine. It declares no marker.
lym-stencil: sha256=4594f01b03e5881b1e7475efb58cf33f074d965d34d6c15228166662818e5ce9 build=935fb407d46b918a520439401a144eb28d3de274 time=2026-11-03T17:43:20Z -->
Run the subagents with the Agent tool, `subagent_type: "general-purpose"`, one call per lens, all in a SINGLE message, as soon as your exploration is done.
A subagent starts with none of your context, so its prompt names, after the boilerplate, the target's files and the fasit's files by path.
The Agent tool returns every subagent's final message in the same turn: never end your turn while a subagent runs.
