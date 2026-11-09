# lym tender pause on an adopt-held run adds no pause of its own

`tender-pause-reuses-adopt-hold` · Note · bug, tender

## Seen

On 2026-11-09, `lym tender pause <slug>` on a run under an adopt hold answered `already_paused: true` and recorded nothing new.
The run started as soon as the adopt check lifted the hold.

## Idea

A pause from another caller records its own entry, so lifting the adopt hold leaves the run paused.