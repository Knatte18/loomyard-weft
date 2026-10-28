# Lingering MSBuild reuse nodes exhaust the user's task limit during .NET runs

`msbuild-node-reuse-task-limit` · Note · bug, shuttle, gate, dotnet, batten

Imported from [issue #885](https://github.com/Knatte18/loomyard/issues/885).

## Seen (Neve, 2026-10-28)

With five loomyard runs and one darn run in the C# sandbox hub (lym-csharp) live, the user slice hit its systemd task limit (user-1000.slice pids.max 3000, pids.current 2997).
Every fork then failed with EAGAIN: three batten runs stopped at Run-Shed or Darn-Write, pushes failed (a git credential helper that cannot fork falls back to a username prompt, reported as an auth failure), and Claude Code's own hooks failed with 'Cannot fork'.
18 idle `dotnet MSBuild.dll /nodemode:1 /nodeReuse:true` processes, about 20 threads each, were left behind by `dotnet build` and `dotnet test` calls made by agents outside the gate; the gate's own runs used `/nodeReuse:false` nodes.
`dotnet build-server shutdown --msbuild` did not end them.

## Workaround

Killed them by hand: `pkill -u <user> -f 'MSBuild.dll .*nodeReuse:true'`, which took the slice from 2940 to 2500 tasks.

## Expected

lym sets `MSBUILDDISABLENODEREUSE=1` (and `DOTNET_CLI_USE_MSBUILD_SERVER=0`) in the environment of every agent session and gate run it starts, so no .NET build an agent runs leaves worker nodes behind.
Beside that, the dobby could hold a new run start while the user slice is near its task limit, since six parallel runs sit close to the default limit even without the leak.