# Gate runs on a LAN machine: Septim takes gate slots through a lym service in WSL2

`remote-gate-farm` · Note · feature, gate, process, hibernate

## Seen

Gate queues on Neve hold landings back: two slots, load near 30 on 12 cores, a tier1 verify past its 30-minute budget.
The operator's desktop Septim (Ryzen 7 9800X3D, water-cooled, Windows 11 with WSL2 installed) sits idle on the LAN.

## Idea

The agents stay on Neve; only gate tier runs move to Septim.

- Septim runs a `lym` gate service inside WSL2 (Linux), started at Windows boot and reachable over SSH on the LAN.
  WSL2's mirrored networking, or a port forward, makes it reachable; the repository and Go build cache live on WSL2's own ext4 disk, not `/mnt/c`.
- The gate slot pool gains Septim's slots beside Neve's local ones; a gate call takes a free remote slot first.
- A gate call syncs the worktree, uncommitted edits included, to Septim over SSH (rsync, so only changes travel after the first copy),
  runs the tier there and streams the `go-json` results back, so the caller sees the same report as a local run.
- When Septim is down or asleep the gate falls back to local slots; Wake-on-LAN is an option.

## Two modes, chosen per hub

- **Stored**, for hubs whose code the operator owns, such as loomyard: the code may be checked out on Septim, with a warm build cache on disk.
- **Ephemeral**, for work code (the OSL-1046 hubs): the worktree is sent into a RAM disk (tmpfs) on Septim, the tier runs there,
  and everything is deleted when the run ends; nothing is written to disk and no git checkout is made.
  The build cache lives only in RAM. The code is processed on Septim but never stored, as with a model server such as Ollama.

## Bounds (operator, 2026-11-28)

- Work code may use Septim only in the ephemeral mode, and only over a closed network; the operator confirmed the network is closed.
- A hub opts in and names its mode; a hub that names none never sends a gate run to Septim.
- Windows itself is not a lym target: the service runs only inside WSL2.

## Open

- Measure first: the same tier1 run on Septim's WSL2 and on Neve.
- How the remote slots are configured, how a remote slot is found to be dead, and what tier2 (tmux) needs there.
- Whether a .NET hub's gate (C#) runs in WSL2 as the Go gate does.
