---
round: 1
focus:
  - "Cards 1-3: does the settings golden test still pass byte for byte after the shell guards move into `internal/shellguard`, and does the no-denied-recovery scan read the table's forms with no form dropped or added?"
  - "Cards 4, 7, 8: do retired keys and the declared `shuttle` to `claude` key move carry old values into `claude.yaml` at every reconcile layer and in run settings, with `deny_python` renamed everywhere it is read?"
  - "Cards 5-6: is `claude.yaml` loaded lazily from told `Dirs`, and do only `NewFromConfig` engines run `Prepare`, `OpenLaunch` and `PromptCacheLifetime` while `claudeengine.New()` keeps its signature?"
  - "Cards 9-10: does reed or the watchdog still hold any Claude-specific knowledge (env keys, namer choice) outside `internal/shuttleengines`, and is the provider tag opaque to reed?"
  - "Per-card mechanical checks and test economy: did each group's type-specific check run against all its targets, and does any new test duplicate an existing one or leave added behavior uncovered?"
---

Focus covers the plan's cards in clusters; `Do not flag` items (plan format, `ImpactSummary`, `DependsOn`) are left out.
