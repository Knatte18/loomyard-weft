# One land notice: fold the beam-PR landing notice into notify_on_land

`one-land-notice` · Note · enhancement, dobby, batten, workspaces

Imported from [issue #93](https://github.com/Knatte18/loomyard/issues/93).

## Seen

In hub `dop`, beam `loomyard-setup` with PR 30 recorded, member task `dotnet10-upgrade` carried `notify_on_land: true`.
Its landing sent the beam orch two separate mails from the beam's dobby:

1. `#130`: "loomyard-setup: landed into the beam while its pull request is recorded: 74fc86c ... (#31); add them to the change description, then run `lym beam describe loomyard-setup`" (the beam-PR landing notice, sent for every landing while a beam PR is recorded).
2. `#131`: "[dobby notice] dotnet10-upgrade: landed as 74fc86c8f712" (the `notify_on_land` line).

Two mechanisms report the same event. The operator found this redundant and confusing: it was not clear which one did what, or that the beam-PR notice exists at all.

## Expected (operator's design)

One mechanism, `notify_on_land`, for every land notice.
While a beam has a recorded pull request, its members land with `notify_on_land` in effect without anyone setting it, and the one line they send carries the beam's follow-up (add the landing to the change description, run `lym beam describe <slug>`).
A task that also has `notify_on_land` set by hand gets that same single line, not a second one.


## Issue comment, 2026-11-09

Operator's refinement: opening the beam's pull request (lym beam pr open) sets notify_on_land: true on the beam's member tasks, and on every member added while the PR is recorded, so dobby has one trigger only: the flag. The beam-PR landing notice as a separate trigger goes away; its follow-up (update the change description, run lym beam describe) rides on the notify_on_land line for members of a beam with a recorded PR.