# Gate runs on a LAN machine: Septim takes gate slots through a lym service in WSL2

`remote-gate-farm` · Note · feature, gate, process, hibernate

## Seen

Gate queues on Neve hold landings back: two slots, load near 30 to 40 on 12 cores, tier1 verifies past their 30-minute budget.
The operator's desktop Septim (Ryzen 7 9800X3D, water-cooled, Windows 11 with WSL2 installed, on cable) sits idle on the home LAN (Wi-Fi 6).
The study `docs/research/gate/septim-farm-study.md` sizes the work and recommends a hand measurement first.

## Idea

The agents stay on the hub's machine (Neve for loomyard, OSL-1046 for work hubs); only Septim's processing power is used, for gate tier runs.

- Septim runs a `lym` gate service inside WSL2 (Linux), started at Windows boot and reachable over SSH on the LAN.
- The gate slot pool gains Septim's slots beside the caller's local ones; a gate call takes a free remote slot first.
  Send `tier1` and `tier2`; keep `tier0` local, since its short runs gain least.
- A gate call sends the worktree, uncommitted edits included, from the hub's machine into a RAM disk (tmpfs) inside Septim's WSL2,
  runs the tier there, streams the `go-json` results back so the caller sees the same report as a local run, and deletes the tree when the run ends.
  No git checkout is made on Septim.
- The build cache is kept in RAM inside WSL2 between runs, so warm runs stay fast.
  A build cache on WSL2's own disk is allowed only if the measurement shows a RAM cache is not enough (operator, 2026-11-28); it stays inside WSL2 like everything else.
- When Septim is down or asleep the gate falls back to local slots.

## One mode for every hub (operator, 2026-11-28)

- Only the mode above is built, for loomyard and work hubs alike; a mode with the code checked out on Septim's disk is not built.
  If it works for loomyard on Neve, it serves OSL-1046 the same way.
- A hub opts in; a hub that does not never sends a gate run to Septim.

## Bounds (operator, 2026-11-28)

- Only over a closed network; the operator confirmed the network is closed.
- Each hub's tree is sent from its own machine: work code from OSL-1046, which alone reaches the work repository.
- The guarantee that matters is that the material never leaks out: everything stays inside Septim's WSL2.
  What stays inside WSL2 is safe, the RAM build cache included, and Windows paging or hibernating WSL's memory is acceptable.
  The design must ensure nothing of the code leaves WSL2: no crash dumps or logs written to the Windows side (such as `%Temp%\wsl-crashes`),
  no telemetry sent off it, no network reach from a tier run beyond the caller, and no Windows drive mounted or Windows program started from a run.
- Windows itself is not a lym target: the service runs only inside WSL2.

## Open

- Measure first: the same tier1 and tier2 runs on Septim's WSL2 and on Neve, cold and warm, with a night left idle.
- Whether OSL-1046 reaches Septim on the LAN, and what its Cortex XDR does to the transfer.
- Whether the work hubs' C# projects build and test on Linux in WSL2.
