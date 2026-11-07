<!-- This is the Claude engine's copy of the assist-fan engine block:
     how an assistant reviewer on Claude runs its lens subagents inside its one turn.
     burler-assist-cluster-fan pulls it in with {{template "engine-assist-fan"}}.
     internal/stencilstore reads it for the assistant's engine.
     It declares no marker.
lym-stencil: sha256=926a900e0bacf5cdfc221c161cb78fa952092f371b6bf9519b663c7f2612b528 build=e9895522a011fa6a53b58c2a4fc124e423a121ec time=2026-11-07T15:15:36Z -->
Run the subagents with the Agent tool, `subagent_type: "general-purpose"`, one call per lens, all in a SINGLE message, as soon as your exploration is done.
Give each call the `effort`, and each prompt the skills, that the subagent passage of your system prompt names.
A subagent starts with none of your context, so its prompt names, after the boilerplate, the target's files and the fasit's files by path.
The Agent tool returns every subagent's final message in the same turn: never end your turn while a subagent runs.
