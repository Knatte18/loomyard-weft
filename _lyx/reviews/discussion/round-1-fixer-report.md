# Fixer report — round 1

All findings fixed in `_lyx/discussion/decision-record.md`; `support-log.md` unchanged.

- F1: D5 now allows only an injected field or option, per instance, and drops the package-var alternative and the `ghAuthTokenTimeout` reference (also parent correction 1).
- F2: D6 gained a Bound: until `test-tiers` decides, plan and card verifies run `-tags tmux` wherever retagged tests ran under `integration` or `smoke`, and no test sits under a tag no gate runs.
  The two matching Open risks were folded into it and removed; an acceptance criterion was added (also parent correction 3).
- F3: D6 date changed to 2026-10-05 (also parent correction 2).
- F4: D2 lists tests that spawn a subprocess under "no coverage", never as redundant; an acceptance criterion was added.
- F5: D9's Bound now limits removal to the teardown that just killed the key, leaves the file alone on a successful connection or a changed socket identity, and states the residual restart race as accepted.
  No separate criterion was added; the existing never-removes-a-live-socket criterion covers the live case.
- F6: D7's guard test also fails on a non-`LookPath` `exec` identifier in `llmkit`.
- F7: D8 no longer gives a background-file count and notes that two retired IDs had none.
- F8: Constraints bullet 1 shortened; the LLM-guard Open risk removed (D7 already states it).

## Deferred

Nothing.
