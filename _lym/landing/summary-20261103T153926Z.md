# A hibernate wake lifts its pause only when the request resumes on wake

A wake from `lym hibernate request` always lifted the hub pause that request added, so every paused run went on at once.
The operator often works on another machine while this one sleeps and does not want runs to go on by themselves after the wake.
This change makes the lift a switch: a hub-wide config key gives the default, off, and a flag on `lym hibernate request` overrides it for one request.

## What changes

- A new hub-wide config module, `hibernate.yaml` in `internal/hibernateconfig`, holds one key, `resume_on_wake`, off in the template.
  It is registered in `configreg` and read through `LoadOrTemplate`, so a hub without the file reads it as off.
- `lym hibernate request` takes `--resume-on-wake`.
  Given either way (`--resume-on-wake` or `--resume-on-wake=false`), the flag decides for that request; otherwise the hub's `resume_on_wake` decides.
- The request file records the choice as `resume_on_wake`, and `lym hibernate status` shows it.
- With the switch on, the wake works as before: the daemon lifts its own pause entry on every path after the wake.
- With the switch off, the daemon still reconciles the hub with origin, holds stopped runs with pair pause entries and sends its notice, but keeps its own hub entry live.
  The request ends `done` with that entry still recorded as owned, so `lym tender resume` or `lym hibernate cancel` lifts it.
  The notice says the hub stays paused and names `lym tender resume`.
- A failure after the wake follows the same switch: with it off, the request fails and the hub entry stays live, and the failure notice says so.
  A failure before the hibernate still lifts the entry whatever the switch says.
- The refusal of `lym hibernate cancel` while the machine is going into hibernation no longer says the daemon lifts its pause on wake.
  It now names `lym hibernate cancel` after the request has ended as the way to lift a pause the wake kept.
- The engine and CLI package docs, `docs/overview.md`, the orch role's hibernate lines and the degrading-callers list in `PATTERN-config-layers` are updated.

## Design decisions

- The key lives in its own package, `internal/hibernateconfig`, not in `hibernateengine`.
  `configreg` has to import the package that declares the template, and `hibernateengine` already depends on `configreg` through its imports, so putting the template in the engine would create an import cycle.
  The engine reads no config: the CLI resolves the flag against the config and passes the result in `SubmitRequest.ResumeOnWake`.
- `resume_on_wake` is classified cross-pair, since lifting a hub pause lets every pair's run go on.
  The key can be set in the hub file, and a worktree's local override is refused.
- The choice is stored on the request, so a later change to the config does not affect a request already waiting.
  A request file written before this change has no such key and reads as off.
- With the switch off, a failure after the wake keeps the pause too, since the operator asked for the hub to stay paused after the wake.

## Known limits

- `lym hibernate request` replaces a finished request without lifting a pause entry that request kept.
  The kept entry then stays live and the new request owns no pause, so after its wake the notice names the standing hub pause and `lym tender resume`.
