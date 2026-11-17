<!-- This is the Claude engine's copy of the fork-spawn engine block:
     the phrase saying how a session spawns a fork that inherits its whole context.
     A stencil pulls it into a sentence with {{template "engine-fork-spawn"}}.
     internal/stencilstore reads it for the launch's engine.
     It declares no marker.
lym-stencil: sha256=b65fbbed8a89117e0725781d679fa4c29070e249008448bfc2c9c6773b70a306 build=40c66912c08839dc8d46483f325e6474c3062a7d time=2026-11-17T14:20:00Z -->
through the Agent tool with `subagent_type: "fork"`, never passing a `name`, since a named fork loses its inherited context
