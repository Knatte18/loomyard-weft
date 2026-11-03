# hibernate: lift the pause on wake only when a switch allows it

A wake from a `lym hibernate request` no longer always lifts the hub-wide pause the request added.
Whether it does is now a switch, and the default is off: the operator often works on another machine while this one sleeps, and runs that start again by themselves after a wake are usually unwanted.
This replaces the lift-always rule `hibernate-wake-leaves-pause` set.

## What changes

- New module `internal/hibernateconfig` with `hibernate.yaml` and one key, `resume_on_wake`, default `false`.
  The module is hub-wide and the key is cross-pair, so the local layer refuses it; it is registered in `configreg`.
  A hub whose base file is absent resolves the template, so a request never fails on a hub not yet reconciled.
- `lym hibernate request` takes `--resume-on-wake`, which overrides the config for that request; `--resume-on-wake=false` keeps the pause whatever the config holds.
  The CLI resolves the choice at request time and stores it on the request as `resume_on_wake`, so the daemon never reads the config itself.
  An unparsable `hibernate.yaml` refuses the request.
- With the switch off, the wake's other steps, its reconcile with origin and its notice included, run as before.
  The pause entry stays live, the request still reads `done`, and the notice to the orch says the hub stays paused and that `lym tender resume` lifts it.
  With it on, behavior is as before: the entry is lifted before the notice.
- The off path keeps the entry live on every path after the wake, a failed wake included, and the failure notice names what became of the entry.
  Failures before the wake still lift the entry.
- `lym hibernate cancel` lifts the pause a finished request left behind, as it already did, and its refusal during hibernation now says so.
- The orch role contract, `docs/overview.md`, the package docs and `pattern/PATTERN-config-layers.md` describe the switch.

## Design decisions

- A kept entry is recorded on the request when the request did not already own one, so a later `cancel` can name it.
- `Submit` refuses over a terminal request whose owned pause entry is still live, with a way forward naming `lym hibernate cancel` and `lym tender resume`.
  Replacing the request would otherwise lose the only record that lets `cancel` lift the entry.
  When the hub cannot be read to make that check, the request is refused too.
- A sleep the machine starts by itself is out of scope: it adds no pause.
  Catching work that moved on another machine meanwhile is a separate check at commit, push and land.

## To check by hand

- Not run: a real hibernate and wake on a Linux machine, with the switch on and off.
  Check that the off path leaves the hub paused, that `lym tender resume` and `lym hibernate cancel` each lift it, and that a new request is refused until one does.
- Finding to settle: `Submit` over a terminal request that owns a live pause entry, in `internal/hibernateengine/verbs.go`.
  The refusal is in place; check that it covers every way the entry is dropped, such as a request that lost its owned record.
- Finding to settle: the failure paths of the switch in `internal/hibernateengine/watch.go`.
  The off-path failure and no-entry notices are pinned in `watch_test.go`; check that no failure path after the wake lifts a kept entry.
