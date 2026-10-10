# Card 5 — reed-orch-form

**Edit:**
- `internal/reedcli#Command`
- `internal/reedcli#reedCLI.addCmd`
- `internal/reedcli#reedCLI.attachCmd`
- `internal/reedcli#reedCLI.listCmd`
- `internal/reedcli#reedCLI.removeCmd`
- `internal/reedcli#reedCLI.resumeCmd`
- `internal/reedcli#reedCLI.statusCmd`
- `internal/reedcli#reedCLI.switchCmd`
- `internal/reedcli#reedCLI.upCmd`
- `internal/reedcli#reedCLI.downCmd`
- `internal/reedcli#reedCLI.watchdogCmd`
- `internal/orchcli#Command`
- `internal/orchcli#orchCLI.startCmd`
- `internal/orchcli#orchCLI.statusCmd`
- `internal/orchcli#orchCLI.refreshCmd`
- `internal/orchcli#orchCLI.distillCmd`
- `internal/orchcli#orchCLI.stopCmd`
- `internal/orchcli#orchCLI.watchCmd`
- `internal/orchcli#orchCLI.resumeContextCmd`

**Uses:**
- `plan:internal/clihelp#AudienceAnnotation`
- `plan:internal/clihelp#AudienceOperator`
- `plan:internal/clihelp#AudienceRole`
- `plan:internal/clihelp#AudienceInternal`

**Intent:** Bring `reed` and `orch` to the D3, D6 and D8 form.
`reed remove <guid> | --name <name>` becomes `remove [<guid>]`, with `--name` documented in `Flags:` and `Long` as the other way to address a strand; every other reed `Use` already fits.
Audiences in reed: `up`, `down`, `list`, `status`, `attach`, `add`, `remove` and `resume` are `operator`; `switch` is `role` (the operator's named example: only the pinned key binding runs it); `watchdog` is `internal` (a daemon lyx spawns).
Audiences in orch: `start`, `status`, `refresh`, `distill` and `stop` are `operator`; the hidden `watch` and `resume-context` are `internal`, as every hidden command must be.
`Short`s already start lower-case and stay; each is read once for the "when to reach for it" rule and reworded only where it names what the verb is rather than when to use it.
Any test asserting the old `remove` usage, found by grepping `internal/reedcli` for `--name <name>`, joins the card.
Coverage for the form is card 16's tree-walk test.

**ImpactSummary:** Annotations and one `Use` string across nineteen command builders in two packages; no runtime behavior changes.
