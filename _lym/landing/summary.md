# Nest loom and tapestry config keys by area and name keys in full

A config key's name now shows its file and the area it governs, one dot per level,
such as `loom.discussion.review_ask_parent`, `loom.code.max_bounces` and `tapestry.plan.max_bounces`.
Until now `loom.yaml` and `tapestry.yaml` spelled the area into flat key names,
and messages and docs often named a key without its file.

## What changes

- `loom.yaml` and `tapestry.yaml` hold one map per area.
  `code_<rest>` becomes `code.<rest>`, and likewise for `discussion`, `split`, `review`, `plan` and `parts`.
- The run-wide review keys that had no `review_` prefix (`reviewer_model`, `fixer_model`, `judge`, `seed`,
  `assist_reviewer_timeout_min`) join `review`.
  `pr_review_*` becomes `pr.review_*`, and `part_writer` becomes `parts.writer`.
- A bare model key that names an area becomes that area's `model`: `discussion`, `raddle`, `friction`, `plan`, `parts`.
  The full tables are in `internal/loomengine/movedkeys.go`.
- Existing values carry over with no operator step.
  Reconcile moves old keys in hub base, worktree and local files and in `profiles.yaml`'s blocks,
  and the hub-wide walk also moves them in board entries' `run_settings`.
  The prime orch is mailed what moved.
- An old name written afterwards is refused, naming both full names and the way forward for that site.
  That covers file loads, `lym config <module> --set`, `run_settings`, profile blocks and recipe setting references.
  `lym config <module> --unset` still removes an old key.
- Verbs outside `lym config` that load the loom or tapestry config now run the after-build reconcile first,
  not only the start verbs.
- Go messages, template comments, contracts, patterns and docs name config keys as `<module>.<dotted path>`.
  Part of the Go comment sweep is deferred, see below.

## Design decisions

- Only `loom.yaml` and `tapestry.yaml` nest. Every other module's keys keep their layout, drift's included.
- `run_settings` stays a module-keyed nested object, and `--set` keeps its form,
  `lym config loom --set code.max_bounces=2`.
  There is no one-token `loom.code.max_bounces` input and no flat dotted `run_settings` key.
- An old name is refused, not read as an alias.
- A move table (`configengine.Spec.MovedKeys`) is separate from retired keys.
  It matches the exact old path, and an old key that is also an area, such as `discussion`,
  matches only a non-mapping value.
- Moves need no marker: a file or board with no old key is left unwritten.
  A null old key, or one whose new path is already set, is dropped and reported.
- `profiles.yaml` and its template base are moved before the template follow,
  so a shipped value the operator changed stays theirs.
- `configengine.KeyName` renders full key names.
  `internal/shellguard` is a declared exception to `PATTERN-config-key-names`:
  it writes the name as a literal, because a leaf package does not import `configengine`.
- `PATTERN-config-layers` changes to cover moved keys, and `PATTERN-config-key-names` is new.
- The `TestLoopIntegration_*` tests that spawn `shed step` now have their own short bound,
  so a stuck step fails with its stderr and status instead of hanging until the package timeout.
  The arming-refusal fixture reconciles its hub before removing the config file,
  so the step no longer reseeds it and starts a real agent.

## To check by hand

- Config keys are still named bare or without their module in many Go comments and package docs.
  Messages, flag help, docs and contracts were fixed;
  the remaining comment sweep is deferred to the board note `config-key-comment-sweep`.
- Check that the moved-keys mail is still sent, and tested, when the walk fails after the hub-wide commit
  (`internal/hubreconcile/walk.go`).
- Check that each site that reconciles before loading loom or tapestry config has a test
  (`loomcli`, `battencli`, `shedcli`, `webstercli`, `shuttleparity` and the other callers).
- Check that the `lym config reconcile` help and the docs describe the new behaviour:
  every reconcile moves renamed keys, and more verbs than the start verbs run the walk.
- Check that no key is still named bare in messages from `gatecli`, `batcher`, `battencli`, `beamcli`,
  `webstercli`, `hibernatecli`, `reportcli`, `preflightshed`, `weaverengine`, `shedarbiter`,
  `internal/sheddriver/tackler.go`, `internal/burlerengine/fix.go` and `internal/loomshed/plancontext.go`.
- A hub whose machines run different builds is exposed until all are upgraded.
  An older build refuses a moved hub base file where an old scalar key is now a map,
  and otherwise warns and runs on template defaults for the moved keys.
  It also prunes the nested keys from its own local files and still accepts old names in `run_settings` and profiles.
- An old-name `run_settings` entry that an older build writes after a walk waits for the next build's walk.
  Seed-Child refuses it meanwhile.
- A loom or tapestry key that lands on `main` flat before this merges must be nested under its area.

## Merge danger

**Door:** one-way
**Blast radius:** config
