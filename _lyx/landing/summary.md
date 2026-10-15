# Engine-owned config, shell-guard table, provider-blind reed

Second step of the Codex-readiness series.
Each engine now owns its settings, the shell guardrails live in one engine-neutral table, and reed no longer carries Claude knowledge.
Claude stays the only registered engine, and a Claude session's hooks, notices, launch line and environment are unchanged.

## What changes

- `claudeengine` owns a layered `claude.yaml` module (template, repo default, worktree, local) with the keys `binary`, `deny_agent_tool`, `deny_ask_user_question`, `prompt_cache_ttl` and `prompt_cache_ttl_roles`.
  `binary` keeps `${env:LYX_SHUTTLE_CLAUDE:-}`.
  `shuttle.yaml` loses every Claude key; its python switch stays as the neutral `deny_python`, renamed by `shuttleengine.MigrateConfig`.
- The new leaf `internal/shellguard` declares every shell-command guard: python, raw go and lyx-owned environment variables, in-place edit, computed-path `rm`, machine sleep.
  `claudeengine` renders the rows as PreToolUse hooks; the Claude tool guards (Agent, AskUserQuestion, webster-in-fork) stay in `claudeengine`.
  `cmd/lyx/norecoverydeny_test.go` reads its denied forms from the table plus its own list of the operator's permission-set forms.
- Engines declare the environment keys to scrub (Claude: `CLAUDECODE`, prefix `CLAUDE_CODE_`).
  reed takes the union as a required constructor input, and `CleanClaudeEnv` is deleted.
- `reedengine.Strand` gains an opaque `provider` tag that shuttle sets in the run and open paths and that reed and shuttle keep through every rewrite.
  The watchdog takes a map from tag to `SessionNamer`; an empty tag maps to the `claude` namer, and a tag with no namer gets pane-title repair only.
- `PATTERN-shell-guard-table` is new; `PATTERN-leaf-packages`, `PATTERN-config-strictness`, `PATTERN-shuttle-provider-seam` and `PATTERN-no-denied-recovery` are updated, and the package docs and `docs/overview.md` follow.

## Design decisions to review

- **Lazy engine config.**
  Engines load `claude.yaml` in the methods that read it (launch, open, `PromptCacheLifetime`), not at construction.
  An invalid file refuses only a run start, open or resume, naming the file and key; stop, teardown, strand removal and state reads never load it.
  `configreg` reaches the module through `shuttleengines`, so the provider-seam scan needs no new exemption.
- **Cross-module key move.**
  `configreg` declares a key-move table (the five old `shuttle.yaml` keys to `claude.yaml`) that `configsync` applies first in every reconcile entry point, per layer, and `PrepareRunSettings` applies to run settings.
  At a base layer a non-default target value wins over the source; at a sparse layer any present target value wins.
  Collisions and overwrites are reported, a dry run writes nothing, and a second pass is a no-op.
  The table stays permanently.
- **Retired keys.**
  `configengine.Spec` gains retired keys.
  Until a layer is reconciled, `shuttle.yaml` loads as if the old keys were absent instead of refusing, so stop and teardown keep working across the deploy.
  A retired unit is judged by its old category alone, so a wrong-shaped TTL value loads in `shuttle.yaml` and is refused later by `claude`'s own local validation.
- **Unreconciled layers.**
  Until the first walk or `lyx config reconcile --apply`, an operator's binary path, switched-off Agent deny, TTL roles and `claude_deny_python: false` are not in force.
  A board entry's run settings naming the old keys are moved only when the task is next seeded.
- **Table payload contract.**
  `shellguard` patterns match a JSON payload with the shell command as a string under `"command"`, as Claude's does.
  Codex's payload may differ; `codex-engine` then adapts rendering or the table.
- **`gateslot` admitted to a leaf.**
  `shellguard` imports `internal/agentname` and `internal/gateslot` so each environment-variable name stays declared once.
  This departs from the `PATTERN-leaf-packages` header clause "never a feature package", and the entry says so.
- **Empty provider tag is Claude, permanently.**
  This keeps the repair of strands recorded before the tag and strands added by `lyx reed add`, the orch above all.

## To check by hand

- The `tmux` and `llm` test tiers were not run during the work; run them before landing.
- `PATTERN.md`: five existing overview entries were trimmed to fit the new entry under `lint-pattern`'s headroom; check that no per-row guardrail comment was lost in the trim.
- The steer texts in the `shellguard` rows and the line-break handling in the test-check texts: a review found a mismatch between the two and left it unverified after the fix; check the rendered steer against its test expectation.
- `creel-core` runs in parallel and constructs shuttle runners and reed engines in `creelcli`; whichever lands second adapts its construction sites to the new `shuttleengines.New` signature and reed's required scrub input.
