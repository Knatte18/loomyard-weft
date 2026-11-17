# lym adopt runs in the foreground over pushed state

All five batches recorded `status: done`, with no departures or renames reported.

## Batch 01 — board-replay-holds (head e877782b)

Bolt replay carries a commit's holds and the machine registry, and refuses an adopt commit over another machine's new hold.
The one-commit adopt write with `--take-hold`, and the adopt-fix and clear setters, landed in boardengine.
Fabric gained the takeover entry's production writer with its rejected-push retry, the origin run read and the hand-over mark's drop.
Deviations (scope guard): `internal/boardengine/holdwrite_integration_test.go`, `replay.go`, `replay_test.go`,
`internal/fabricengine/destroy.go`, `export_test.go`, `takeoverrun_integration_test.go`, `takeoverwrite.go`.

## Batch 04 — shellguard-take-hold (head feab3af8)

A shell-guard row denies `lym adopt --take-hold` to agents.
The orch watcher and the dobby read this machine's Bolt hold, and `lym beam status` shows it.
Deviations: `internal/beamcli/dobbydeps_integration_test.go`, `internal/orchcli/watch.go`.

## Batch 07 — batten-hub-run-slugs (head 31f8630b)

`battencli.HubRunSlugs` gives the hub-wide run list for the catch-up.
The adopt check's ends keep try counts in the hold and clear it on green.
Deviations: `internal/adoptcli/check_test.go`, `fixloop.go`, `fixloop_test.go`, `green_test.go`, `holdcheck.go`, `holdcheck_test.go`,
`internal/battencli/catchup_integration_test.go`.

## Batch 09 — adopt-foreground (head 6cf07256)

`lym adopt` runs every step in the caller's process through a foreground runner, with `--take-hold` added.
The detached worker, the adopt record package and the adopt pause, plus `--keep-local`, `--drop-local` and `--discard`, are gone.
Deviations: `internal/adoptcli/adopt.go`, `adopt_integration_test.go`, `adopt_test.go`, `caller_test.go`, `deps.go`, `foreground.go`, `refusal.go`.

## Batch 10 — retired-adopt-wording (head 0a19ae29)

Ways forward and docs no longer name the retired flags and pause.
No deviations.
