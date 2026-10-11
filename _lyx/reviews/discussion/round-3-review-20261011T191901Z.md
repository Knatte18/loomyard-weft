---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:62
    summary: D4 opens landing's require_pr_to_base, squash and co_authored_by and darn's review_max_bounces to the local layer, so a pair's untracked file can skip PR approval and Darn-Review and reshape the shared parent branch, against D4's own bound
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:73
    summary: D4's rationale and R3 rest on a false premise, since the git-ignored per-worktree .env already binds the env markers of reed.tmux, shell, tmux_conf, mouse, debug_log and shuttle.claude, run_dir untracked and persistently in a pair
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:91
    summary: D6 never says the strip compares unexpanded values, and an env-expanded comparison strips a literal equal to the walk's expansion of a marker below, changing the worktree's resolution in any other environment
  - id: F4
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:93
    summary: D6's claim and the AC that migration leaves every worktree's resolution unchanged miss the keys the seeded default adds under a worktree whose file lacks them, such as a seed-only alias or lens the prime holds
---

### [BLOCKING] Hub-wide `LocalKeys` fail D4's own category test

**Location:** `_lyx/discussion/decision-record.md:62` (D4's `LocalKeys` lists), against the Bound at line 70 and the Rationale at line 73.

**Issue:** D4 says "the lists come from testing every key of every module against these categories" and bounds the local layer to never "switch off an agent guard or [be] a fact shared across pairs".
That test was not applied to the hub-wide keys it opens:

- `landing.require_pr_to_base`: with `[]` in a pair's `landing.local.yaml`, `Publish.Call` returns Done before any push or GitHub call (`internal/landingshed/publish.go:118`), `PRGate.Call` returns Done with no approval check (`internal/landingshed/prgate.go:56`), and Finalize merges and pushes into the parent branch.
  It switches off the operator's PR approval gate on a landing into a branch every pair shares.
- `landing.squash` and `landing.co_authored_by`: they set the merge shape and trailer of the commit Finalize writes on the shared parent branch (`internal/landingshed/finalize.go:194`), a fact shared across pairs.
- `darn.review_max_bounces`: `0` "runs neither the review nor Describe and goes from Darn-Write straight to Publish" (`internal/darnengine/template.yaml`); loom refuses a budget below 1, darn does not, so the open "budgets" of the Bound include a review switch-off.

Combined, a file a pair's agents can write (R3), and that landing does not show, lands unreviewed, unapproved code on `main`.
D4's own Rationale says hub-wide modules are hub-wide "because a task branch must not change them", yet it opens nearly every landing key.
Under the rubric's attack-surface rule this is a bypassable guard with a false stated bound.

**Fix:** Re-run D4's category test over every hub-wide key and add a category for a key that gates or shapes a landing into the parent branch.
Take `require_pr_to_base`, `squash` and `co_authored_by` out of landing's `LocalKeys`, which leaves its model picks and timeouts.
Either take `review_max_bounces` out of darn's `LocalKeys` or refuse a local value below 1.
Add acceptance rows naming `landing.require_pr_to_base` and a local `darn.review_max_bounces: 0` among the refused keys.

### [MEDIUM] D4's rationale and R3 ignore the per-worktree `.env`

**Location:** `_lyx/discussion/decision-record.md:73` (D4 Rationale) and `:171` (R3).

**Issue:** The Rationale says the denied per-worktree keys "persist in a pair only through a tracked file that landing flags, since the `${env:...}` markers some of them carry bind only the process that sets the variable".
That is false.
`configengine.load` expands markers through `envsource.Build(baseDir)` (`internal/configengine/config.go:168`), which reads `<anchor>/.env` before overlaying the process environment.
`docs/overview.md` lists `.env` as a git-ignored, per-worktree file.
So a pair's `.env` already binds `reed.tmux`, `reed.shell`, `reed.tmux_conf`, `reed.mouse`, `reed.debug_log`, `shuttle.claude` and `shuttle.run_dir`, every denied key but the `claude_deny_*` booleans, which carry no marker.
That file persists in the pair, reaches every later load, is writable by the pair's agents and is invisible at landing, the exact properties the Rationale assigns only to the new local file.
R3's "D4's lists bound what it gains to session tuning" is false for the same reason: an agent writing `LYX_SHUTTLE_CLAUDE=...` into the pair's `.env` sets the executable.
The deny lists stay correct, since the local layer should not add a second route; the premise and the stated bound are what fail.
This premise is the round-2 F3 fix's rewrite, which looked at the process environment and missed `.env`.

**Fix:** Rewrite the Rationale so the deny lists rest on the local layer adding no new untracked route, not on there being none today.
State in R3 that `.env` stays an untracked route to every marker-carrying denied key, and leave closing it to the parent, or bring it into scope as its own decision.

### [MEDIUM] D6's strip does not fix how markers are compared

**Location:** `_lyx/discussion/decision-record.md:91-93` (D6) and `:188` (acceptance criterion).

**Issue:** D6 strips "every key whose value equals what the layers below resolve to" but does not say whether the comparison is over the raw file text or over values after `${env:...}` expansion.
Expansion depends on the process environment and on each worktree's own `.env`, so an expanded comparison is environment-dependent.
Example: the seeded default holds `shell: ${env:LYX_REED_SHELL:-bash}`, a worktree's tracked file holds the literal `shell: /bin/zsh`, and the walk runs with `LYX_REED_SHELL=/bin/zsh` exported in the operator's profile, as the template's own `tmux_conf` comment recommends for that key.
The values compare equal, the literal is stripped, and any later process without that variable resolves the worktree's shell to `bash`.
The same holds for a marker in the worktree file against a literal below.
This breaks the "never changes what a worktree resolves" claim for marker-carrying keys, which include the executables.

**Fix:** State in D6 that the strip compares unexpanded nodes, a marker equal only to the same marker text, and add an acceptance row with a worktree literal that equals the walk's expansion of a marker below and is kept.

### [LOW] Migration adds keys under a worktree whose file lacks them

**Location:** `_lyx/discussion/decision-record.md:93` (D6) and `:188` (acceptance criterion).

**Issue:** D6 claims the strip never changes what a worktree resolves, and the acceptance criterion says every migrated worktree resolves every module "to the same values as before".
The comparison point "before" is a worktree with no default layer.
Seeding the default puts every key of the prime's file beneath every worktree, and D1 lets no layer delete a key below.
A worktree whose file lacks a key the prime's holds, outside the template, therefore resolves to the prime's value after migration.
The live case is the seed-only modules, which reconcile never fills: a pair that removed the prime's redefined `sonnet` alias from its `models.yaml` resolved the template's alias before, and resolves the prime's after.
A lens or fan the prime added after the pair forked appears in the pair the same way.
The strip cannot repair it, since it only removes keys.

**Fix:** Narrow D6's claim and the acceptance criterion to keys the worktree file holds, and state the additive case as a known change.
Or have the walk report each key the default adds beneath a worktree, beside `stripped`, so the operator can pin it back with `lyx config <module> --set`.

## Focus departures

None of the three directives was departed from; each verdict is stated below for the record.

- Directive 1, every open key of every per-worktree module: no defect among them.
  reed's `width`, `height`, `collapsed_rows`, `min_full_rows`, `watchdog`, `operator_quiet_s`, `selvage.height_rows` and `segment_colors` act on the pair's own session, through `new-session -x/-y`, its own window hooks and its own resize watch, never a `-g` server option.
  shuttle's open keys are its own run's timeouts, polls and cache TTLs; `startup_timeout_s` shrinks only this worktree's orphan-sweep age guard, and `run_dir` stays denied.
  logger's retention keys sweep only `<anchor>/.lyx/logs`; loom, webster, orch, batcher, models and burler are model picks, budgets and timeouts.
  The directive's question is settled for per-worktree modules, and F1 records the same test failing for hub-wide `LocalKeys`, which D4's Bound also covers.
- Directive 2: open maps compared as one unit hold, so a worktree whose open map differs in one entry keeps it whole and resolves the same; F3 and F4 record the two cases where resolution still changes.
- Directive 3: F2.
