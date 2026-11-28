# Gate runs on a LAN machine: Septim takes gate slots through a lym service in WSL2

`remote-gate-farm` · Note · feature, gate, process, hibernate

## Seen

Gate queues on Neve hold landings back: two slots, load near 30 on 12 cores, a tier1 verify past its 30-minute budget.
The operator's desktop Septim (Ryzen 7 9800X3D, water-cooled, Windows 11 with WSL2 installed) sits idle on the LAN.
The study `docs/research/gate/septim-farm-study.md` sizes both modes below and recommends a hand measurement first.

## Idea

The agents stay on the hub's machine; only gate tier runs move to Septim.

- Septim runs a `lym` gate service inside WSL2 (Linux), started at Windows boot and reachable over SSH on the LAN.
  WSL2's mirrored networking, or a port forward, makes it reachable; the repository and Go build cache live on WSL2's own ext4 disk, not `/mnt/c`.
- The gate slot pool gains Septim's slots beside the caller's local ones; a gate call takes a free remote slot first.
- A gate call syncs the worktree, uncommitted edits included, to Septim over SSH (rsync, so only changes travel after the first copy),
  runs the tier there and streams the `go-json` results back, so the caller sees the same report as a local run.
- When Septim is down or asleep the gate falls back to local slots; Wake-on-LAN is an option.

## Two modes, chosen per hub

- **Stored**, for hubs whose code the operator owns, such as loomyard on Neve: the code may be checked out on Septim, with a warm build cache on disk.
- **Ephemeral**, for work code (the hubs on OSL-1046): the worktree is sent from OSL-1046 into a RAM disk (tmpfs) inside Septim's WSL2,
  the tier runs there, and the tree is deleted when the run ends; no git checkout is made.
  The code is processed on Septim but never checked out there, as with a model server such as Ollama.

## Bounds (operator, 2026-11-28)

- Work code may use Septim only in the ephemeral mode, and only over a closed network; the operator confirmed the network is closed.
- The caller for work code is OSL-1046 itself: only it reaches the work repository, so the tree is sent from OSL-1046 to Septim, never from Neve.
- For work code the guarantee that matters is that the material never leaks out: everything stays inside Septim's WSL2.
  What stays inside WSL2 is safe, a build cache kept in RAM between runs included, and Windows paging or hibernating WSL's memory is acceptable.
  The design must ensure nothing of the work code leaves WSL2: no crash dumps or logs written to the Windows side (such as `%Temp%\wsl-crashes`),
  no telemetry sent off it, no network reach from a tier run beyond the caller, and no Windows drive mounted or Windows program started from a run.
- A hub opts in and names its mode; a hub that names none never sends a gate run to Septim.
- Windows itself is not a lym target: the service runs only inside WSL2.

## Open

- Measure first: the same tier1 and tier2 runs on Septim's WSL2 and on Neve, cold and warm, with a night left idle.
- Whether OSL-1046 reaches the closed LAN and Septim, and what its Cortex XDR does to the sync.
- Whether a .NET hub's gate (C#) runs in WSL2 as the Go gate does.
