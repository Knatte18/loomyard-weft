# creel: no delivery into a collapsed (too short) strand pane, and no reason surfaced

`creel-collapsed-pane-delivery` · Note · bug, creel, reed, shuttle

Imported from [issue #658](https://github.com/Knatte18/loomyard/issues/658).

Found by the creel stage 1 audit (board note `creel-test`), in a sandbox hub, with `lyx` built from `main` at e131c7625.

## What was seen

Creel delivers nothing into a strand whose pane reed has collapsed, and says nothing about why.
Reed stacks hand-added strands of one worktree in one window; every strand but the bottom one collapses to `collapsed_rows` (3 rows).
A 3-row Claude pane cannot draw its input box, so shuttle's idle probe answers not idle with `TooShort` set.
Creel's delivery adapter (`internal/creelcli/delivery.go`) drops `TooShort`, and the scheduler simply keeps probing.
Mail is never reminded, and an inject sits until its TTL runs out and is journaled `expired`, because the TTL runs while the strand is live and unpaused.
The orch watcher, by contrast, records `orch pane too short for the idle probe; resize or use the larger client`.

## Steps

1. In a sandbox hub add three strands to the prime: `lyx reed add --role bob --cmd claude`, then `--role carol --cmd bash`, then `--role dave --cmd bash`.
2. bob's pane is now 220x3 (`tmux list-panes`).
3. `lyx creel inject --wait --timeout 120s ctest:bob "Reply INJECTED"`: times out (exit 4); bob's pane shows nothing.
4. The journal shows `aged` records every minute and, ten minutes later, `expired` for that inject.

## Expected

Either creel can deliver into a collapsed strand (for example by having reed or shuttle make the pane tall enough for the send), or the stall is surfaced: a `lyx creel inject --wait` / `log` / `inbox` line that names the short pane as the reason, so a sender is not left guessing.
