# Orch remote control is off by default

lym turned on Claude Code's remote control for every orch it launched, because `internal/orchengine/template.yaml` shipped `remote_control: true`.
The operator decides when a session gets remote control, so lym must never turn it on by itself.

## What changes

- `remote_control` in the orch template is now `false`.
  A hub whose `orch.yaml` sets no value takes the new default on every launch and resume.
  The RC button of a pane still turns remote control on for one agent.
- `orchengine` gains `FlagRemoteControlDefault`, registered on the `orch` row of `configreg` as the reinterpretation `orch-remote-control-off-by-default`.
  A hub `orch.yaml` that sets no `remote_control` is flagged once, with `lym config orch --set remote_control=true` as the way to keep it on.
  The file is left unchanged, since lym does not choose the operator's value for them.
  A file that sets the key, on or off, is neither changed nor flagged.
- A reed resume or absent-pane relaunch replays the strand's stored command line, which still carried `--remote-control` from before the default changed.
  `reedengine.Engine` gains `SetRevivalFilter`, and `revivalCmd` applies it in `Resume` and in the relaunch of `AddStrandUnless`.
  `orchengine.RevivalFilter` strips the `--remote-control <name>` pair from the stored line while `orch.yaml` has `remote_control` off.
  It reads `orch.yaml` on each call, and drops the argument when the file does not load.
  `reedcli` and `battencli` install it when they build the reed engine.
- Other launch paths: the diff changes none; the revival paths above were the only ones replaying a stored line.

## Design decisions

- No transition window: a revived orch strand loses remote control at once when the hub's config has it off, rather than keeping it until the strand is next created fresh.
- The stripping is a regex over the stored line, which handles the POSIX-quoted, PowerShell doubled-quote and bare-name forms of the name.
  A name quoted in a shape those do not cover would survive the strip.
- A config reinterpretation is flag-only: it reports and does not rewrite, so a hub's file is never changed by the update.

## To check by hand

- The stored resume line keeps `--remote-control` in `internal/reedengine/lifecycle.go` (`Resume`) and `strand.go` (relaunch).
  The revival filter drops it while `orch.yaml` has remote control off.
  Bring up an orch with `lym beam start` on a hub with no `remote_control` set, then resume a strand stored before the change, and confirm the pane's command line has no `--remote-control`.
- On a hub whose `orch.yaml` sets no `remote_control`, confirm the reinterpretation notice appears once and names the `--set` command.

## Merge danger

**Door:** one-way
**Blast radius:** orch
