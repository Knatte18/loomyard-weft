# Nest loom and tapestry config keys by area, move old names on reconcile, and name every key in full

`config-keys-nested` · Task · enhancement, config, loom, reconcile, profiles, shed, batten, fabric, drift, landing, gate, webster, shuttle, dobby, claude, codex, stencil, pattern, docs

Decision record: [Sketch](decision-config-keys-nested.md)

Depends on: [`parent-review-stale-window`](design-parent-review-stale-window.md), [`webster-amend-stop-no-notice`](design-webster-amend-stop-no-notice.md), [`async-run-pusher`](design-async-run-pusher.md), [`pr-review-dead-session-unseen`](design-pr-review-dead-session-unseen.md), [`fixed-blocking-escalates-parent`](design-fixed-blocking-escalates-parent.md), [`red-first-csharp`](design-red-first-csharp.md)

Filed from the Sketch `config-keys-nested-by-area`.

Takes: the Sketch's decision record; the flat loom and tapestry templates and their readers; configengine's dotted-path layering, configsync's reconcile, hubreconcile's walk and EnsureOrWarn, configreg.PrepareRunSettings and boardengine's upsert path
Delivers: loom.yaml and tapestry.yaml nested one map per area with every reader on the nested paths, pr_review_parent_timeout_min (read by autonomy mode) included as pr.review_parent_timeout_min; a move table that every reconcile, the profiles follow and each hub-wide walk apply to file layers, profiles.yaml blocks and board run_settings; refusal of an old name at every site with that site's way forward; EnsureOrWarn before every non-config verb that loads loom or tapestry config; every key named in full in Go messages, template comments, contracts, pattern and docs outside docs/research; PATTERN-config-layers changed and PATTERN-config-key-names added
Exports: configengine.KeyName, configengine.Spec.MovedKeys, the configreg helper that moves a module-keyed settings object, the helper that nests a dotted path into a run_settings or profile object

The part changes these packages:

- internal/loomengine (about 450 changed lines)
- internal/reviewbudget (about 120 changed lines)
- internal/configengine (about 300 changed lines)
- internal/configsync (about 350 changed lines)
- internal/hubreconcile (about 300 changed lines)
- internal/configreg (about 250 changed lines)
- internal/configcli (about 40 changed lines)
- internal/runprofile (about 120 changed lines)
- internal/beamcli (about 50 changed lines)
- internal/shedrecipe (about 100 changed lines)
- internal/loomrecipe (about 80 changed lines)
- internal/loomcli (about 300 changed lines)
- internal/loomshed (about 20 changed lines)
- internal/shedadapters (about 60 changed lines)
- internal/battencli (about 80 changed lines)
- internal/hardenshed (about 20 changed lines)
- internal/fabricengine (about 40 changed lines)
- internal/fabriccli (about 40 changed lines)
- internal/driftengine (about 40 changed lines)
- internal/landingshed (about 30 changed lines)
- internal/gateslot (about 30 changed lines)
- internal/webstercli (about 20 changed lines)
- internal/shuttleparity (about 20 changed lines)
- internal/preflightshed (about 10 changed lines)
- internal/dobbyengine (about 20 changed lines)
- internal/claudeengine (about 10 changed lines)
- internal/codexengine (about 10 changed lines)
- contracts (about 200 changed lines)
- docs (about 80 changed lines)
- pattern (about 60 changed lines)
