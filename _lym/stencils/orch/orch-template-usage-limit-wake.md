<!-- This is the wake line the `lym orch` watcher types into the orch once a usage limit that stopped it should have reset, rendered by RenderUsageLimitWake (internal/orchengine/prompt.go).
     It must render to ONE line: shuttle's Send refuses multi-line text.
     Its one marker is {{.reset}}, when the limit reset, or that its reset time could not be read.
lym-stencil: sha256=1d21c0ff03ab5f166739fcbc2fcbd1cdc6db5b9fd98d6d8b95ec69ffe0aa38d2 build=563886346f0323010a227d327791e0266fe49630 time=2026-11-01T16:22:17Z -->
[lym orch watcher] The usage limit that stopped this session {{.reset}}. Run `lym tender revive`, then wait for the notices of the runs that stopped; read a filtered `lym batten status <slug>` only for a stop a notice names, and handle that stop as your role says.
