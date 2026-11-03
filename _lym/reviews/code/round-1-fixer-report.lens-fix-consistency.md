```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix set for agreement with its siblings (lens fix-consistency).

**Findings**

1. **LOW** — `internal/pairteardown/doc.go:78` (twin of the changed `internal/pairteardown/claim.go:102`).
   - The doc says the settle "only warns, naming `lym batten status <slug>` and `lym batten pause <slug>`".
   - The warning now also names `lym tender pause <slug>`, so the package doc no longer matches the message.
   - Evidence: `claim.go:102` now carries `...or with \`lym tender pause %s\` when its work already reached its parent`. The doc line is unchanged.

2. **LOW** — `internal/battencli/wire.go:1486` (`PauseWayForward: "lym batten pause %s"`).
   - It is another operator-facing pointer to `lym batten pause` as the way to hold a run. F8 fixed three such places, and this one was left as it was.
   - I did not check where `PauseWayForward` is consumed. If it reaches an operator for a run that may already have reached its parent, it is a twin of the F8 sites. The `pairteardown/doc.go` item above is the clearer one.

3. **LOW** — `internal/battencli/cli.go:355`, the `goto` help (the `lym batten pause some-slug` example near line 355 sits in a neighbouring verb's help).
   - F6 says `lym batten goto` does not hold a reached-parent run, and the fix put that only in `orch-role.md`.
   - The `pause` help now states the exception, but the `goto` help carries no matching note. I did not read the `goto` text itself, so this is unconfirmed.

**Checked and consistent**
- `internal/dobbyengine/doc.go:70` now agrees with the lines under it and with `drivable`.
- The `orch-role.md` rule 2 exception, rule 3 sentence and the `pause` help all name `lym tender pause`.
- The `PausedReachedParentOwnedElsewhereProbeGivesFull` and `PausedReachedParentHeldGivesNone` rows match `planPass`: the `Held` check comes first, and the owned-elsewhere probe tick is uncovered.
- The `DobbyRuns` doc comment's "one Warn" still holds as one Warn per episode.

**Tool calls**
1. `grep -rn "batten pause"` over `.go` and `.md` files, excluding tests and `_lym`. My context held only the three files F8 named, so I needed the repo-wide twins.
