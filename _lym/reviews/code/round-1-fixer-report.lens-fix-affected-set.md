```yaml
lens: fix-affected-set
outcome: reported
```

Task: affected-set check of the fix (reran the searches behind F1, F2, F4 and F5).

Unchecked findings: F3 and F6. Neither rests on a set that a search can rerun. F7 is `decision`-class and outside the fix.

Findings:

- **LOW**, `internal/beamcli/dobbydeps.go`, `tellRefusal`'s warning `"beamcli: a refused autonomy setting was not posted to the orch"`: F1 fixed the mail to say `beam.autonomy`, but this twin message in the same function still calls the key "autonomy setting".
  - Evidence: `PATTERN.md:17`, `PATTERN-config-key-names`: "Naming a config key in a message, comment, doc or stencil: in full, `<module>.<dotted path>`, rendered in Go by `configengine`'s one key-name helper".
  - The branch added this line (it is a `+` line in the branch diff), and `autonomyName()` now exists to render it.

Checked and complete:

- **F4:** the `.On()` readers are `loomcli/driftwiring.go:102`, `loomcli/pawlwiring.go:154` and `slackcli/wiring.go:131`. Only the first has an `unattended` row, which is the gap the review's `fix` already names.
- **F5:** every caller of `AutonomyWithRefusal` takes the new three-value return, and no "next read that is not refused" wording remains under `internal` or `_lym/landing`.
- **F2:** the non-test callers of `dobbyengine.LoadConfig` are the dobby, the view reader (`beamcli/cli.go:442`, `battencli/overview.go:628`), adopt's fix loop (reached from `adoptcli/holdcheck.go:302`) and `configreg`'s `CheckLoad`. The rewritten blast radius names all four.

Tool calls (4):

1. Searched the added non-test lines naming autonomy in a string, because my context lacked the branch-wide set of messages for F1.
2. Grepped `.On()` and `AutonomyOn` for F4's reader set, which my context did not hold.
3. Grepped the callers of the beam config load for F2, since my context had only part of them.
4. Grepped references to the changed signature and the old wording for F5, to find sites my context had not shown. The same output showed no other stale site.

A fifth call read the `PATTERN-config-key-names` text in `PATTERN.md`, which my context lacked (5 calls in total).
