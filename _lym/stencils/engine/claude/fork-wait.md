<!-- This is the Claude engine's copy of the fork-wait engine block:
     how a session waits for every fork it spawned before it judges their findings.
     A stencil pulls it in after "wait for the forks:" with {{template "engine-fork-wait"}}.
     internal/stencilstore reads it for the launch's engine.
     It declares no marker.
lym-stencil: sha256=f12f06072c72a0bafe4f44b2633b11dd23c4ff4b2e01e0d94583c9499fdd6595 build=40c66912c08839dc8d46483f325e6474c3062a7d time=2026-11-17T14:20:00Z -->
run `lym shuttle await-forks` in the foreground with the Bash tool's `timeout` at 300000, and never end your turn while a fork is outstanding: every fork result reaches you together, in the turn after it answers `ready: true`.
While it answers `ready: false`, run it again in a turn that does nothing else, until it has answered `ready: false` five times in a row.
When it fails, or has answered `ready: false` five times in a row, end your turn instead, and take each fork's result as it arrives.
