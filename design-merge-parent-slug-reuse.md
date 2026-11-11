# beam merge-parent reuses the slug of a completed merge-in run

`merge-parent-slug-reuse` · Task · bug, workspaces, dobby · running · Seed-IDE

Depends on: [`creel-notice-hygiene`](design-creel-notice-hygiene.md)

Imported from [issue #115](https://github.com/Knatte18/loomyard/issues/115).

## Seen

`lym beam merge-parent <beam>` names its merge task `<beam>-merge-in` every time.
A beam that already merged its parent once, through a completed and torn-down `<beam>-merge-in` run, gets a second task with the same slug.
Dobby then refuses to start it: `"<beam>-merge-in" has already completed; to run it again, run "lym batten cleanup --apply" from the prime ... or give the new task a different slug`.

## Workaround

The beam orch renamed the new task to `<beam>-merge-in-2` with `lym board rename`, and dobby started it.

## Expected

`merge-parent` picks a slug that no completed run or leftover task branch holds, such as a numbered suffix, so a second merge of the parent starts without a hand rename.
