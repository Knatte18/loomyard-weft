```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of the F2–F4 change set against its surroundings.

1. **MEDIUM**: the new re-step sentence in `internal/loomengine/anomaly.go` (`AnomalyTransientStop` doc), the `internal/loomcli/doc.go` transient-stop paragraph and `friction-template-reflection.md` ("the driver also re-steps a git-transport or GitHub API stop with waits, and any other class goes to the tackler").
   - This holds for the Go driver: `internal/sheddriver/doc.go` rule table and `driverules.go:289-294`.
   - It does not hold for the prompt-driven driver. `contracts/stencils/shed/shed-template-driver.md` has no network waits and no tackler step for this case. It says that after the loop's re-step a transient `kind` of any class gets "the one fork below under the repair cap, as for a `producer` error".
   - `internal/loomcli/driverprompt.go:21` still names that stencil. I did not check which configuration launches it.
   - The reflection agent can run after either driver, so the three texts should either name the Go driver or say only that the driver handles what outlasts the loop's re-step.

2. **LOW**: F3's twin is unchanged in `_lym/landing/summary.md`, under `## Changes`.
   - It still says "a nine-hour gap like the one in the report shows whether the lock was held that long or the host was suspended".
   - The `ensure` doc comment now says a gap only "points to" a suspend, on some platforms, or to a wall-clock step.
   - The summary is the change description, a run record. If it is outside the fixer's write surface, the fixer-report should name it.

Everything else agrees:
- The F4 row matches the `ClassifyHalt` doc's precedence.
- The inline `Round(0)` comment is consistent with the new `ensure` doc.
- My search outside `_lym`, tests and research docs found no other list of anomaly kinds that needed the change.

Tool calls:
1. Grepped for the anomaly kinds, the suspend claim and the re-step wording across `.md` and `.go` files, because my context had no repository-wide view of sibling mentions.
2. Read part of `shed-template-driver.md` and the `internal/sheddriver/doc.go` header, and grepped for the stencil's name, because none of these were in my context.
