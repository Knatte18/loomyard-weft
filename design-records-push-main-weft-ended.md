# Seed-IDE fails: records push calls main-weft ended on origin

`records-push-main-weft-ended` · Task · bug, fabric, batten · running · Record-Write

## Seen

Twice on 2026-11-18 a newly started batten run failed at Seed-IDE, right after Open-Window, with:

```
fabricengine: push records side at /home/knatte/Code/loomyard-LYMHUB/loomyard-weft: the entry of branch main-weft is ended on origin, so the branch was not pushed and is left to its teardown
```

It hit `retro-stencil-fixes` in the morning and `shuttle-start-idle-and-prompt` at 09:01Z, while other runs released in the same batch started cleanly.
The branch it names is the prime records branch `main-weft`, which is never ended; the message comes from `recordsSideNotLanded` in `internal/fabricengine/pushanchored.go`.

## Workaround

The orch reran `lym batten run <slug>`; the run then went on through Run-Shed.

## Expected

A push of the prime records side never classifies `main-weft` as ended on origin, and a transient classification does not fail the run at Seed-IDE.
