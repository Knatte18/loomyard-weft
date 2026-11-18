# Adopt in the foreground over pushed state, with hub stop and start

A run's records now reach origin before the run acts on them,
so a second machine can take a hub over from what is pushed.
Bolt records which machine is which and which adopt holds stand,
and every machine catches up from origin before it acts after an absence.
`lym adopt <subject>...` runs every step in the caller's process over that pushed state,
and a takeover entry in a run's own records hands the run over.
The dobby tears down landed and cancelled pairs and reports kept and orphaned ones.
`lym beam stop --all` takes a hub down on this machine behind a stop button,
and `lym machine start` brings back every hub the machine owns, from desktop launchers.
The board prints the prime as the root beam `root`.

## Changes

- 595349b9f675 Push a transition's records before the run acts; hand over on takeover
- 6005496ff858 Machine registry and adopt holds in Bolt, catch-up after absence
- eecdd1b2d95f lym adopt runs in the foreground over pushed state
- f06f074f690a Print the prime as the root beam `root` on the board
- 80ead5844571 The dobby tears down or reports leftover pairs
- 7f2ee87f4e88 beam: `lym beam stop --all` takes the hub down on this machine
- 0c013afa56d0 lym machine start: bring back every hub this machine owns
