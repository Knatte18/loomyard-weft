# Gate settings per machine

`gate-machine-local-config` · Note · enhancement, config

## Seen (2026-10-20, `go_parallel` sweep on OSL-1046)

Every gate key is hub-wide: `lyx config gate --layer local` refuses `go_parallel`, and gate has no worktree layer.
The only layer that takes it is the default one on Bolt, so the sweep's values 8 and 12 reached Neve too while it ran on OSL-1046.
The two machines differ in what bounds a run: Cortex XDR holds new binaries on OSL-1046, and Neve has no endpoint security.

## Idea

`go_parallel`, and possibly `slots`, take a machine-local override that a run on that machine reads and Bolt never carries.
