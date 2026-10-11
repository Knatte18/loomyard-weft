---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:79
    summary: loom's discussion_fan and plan_fan pick the lens set a review segment's reviewer is given, which meets D4's own category test, yet loom's LocalDenied lists only parent_review_wait_min
    fix: In D4 (decision-record.md:79), add `discussion_fan` and `plan_fan` to loom's `LocalDenied` list, and extend the loom sentence at line 81 with the reason, that each picks the lens set a review segment's forks are given, so a local empty value turns a hub-chosen fanned review back to solo untracked. Add `loom.plan_fan` to the LocalDenied examples in the acceptance row at line 229. Leave `discussion_producer` open and say why in the same sentence, that it picks the writer, not a reviewer, if that is the intent.
  - id: F2
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:218
    summary: R3 and D4's rationale name the pair's .env as the untracked route to marker-carrying denied keys, but the process environment wins over .env and the hub's shared tmux server hands its boot environment to every pane in every pair
    fix: In R3 (decision-record.md:218), after the `.env` sentence, add that `envsource.Build` lets the process environment override `.env`, so a `LYX_*` variable in the environment of the process that boots the hub's shared tmux server reaches every later pane of every pair, and that this cross-pair route belongs to the same parent decision (Q12). Mirror the one clause in support-log.md's Q12.
---

### [MEDIUM] loom's fan keys meet D4's lens-set category but stay open

**Location:** `_lyx/discussion/decision-record.md:79`, with the category test at line 83 and the bound at line 87.

**Issue:** D4's category test closes "the prose or lens set a reviewer is given" to the local layer, and it says it asks what a key's value can do, not what kind of key it is.
`loom.discussion_fan` and `loom.plan_fan` choose that lens set.
`internal/loomcli/wiring.go:786-789` hands each fan name to its segment's Bouncer and Burler rows, and `fannedReview` in `internal/loomengine/review.go:151` returns no fan for an empty value.
A repo default that sets `plan_fan: <fan>` is a hub decision to review plans with a lens fan.
A local `plan_fan: ""` drops every Plan-Review round in the pair back to a solo review with no tracked trace, and landing shows nothing (R3).
That is the same untracked-review-weakening D4 closed for `burler.lenses` and `burler.fans` in round 4.
A fan name can only name a fan from the tracked `burler.yaml`, so the key cannot inject prose, and a solo review is still a full review.
That makes this a weakening rather than a hollowing, hence MEDIUM, not BLOCKING.
Still, D4 states one test, and loom's list as written gives a different answer from that test.
The fix is either to deny the two keys or to narrow the category's wording so it plainly excludes a choice among tracked fans.
Denying them is the smaller edit and matches the bound's "hollows ... a review".

### [LOW] The env-marker route is wider than the pair's `.env`

**Location:** `_lyx/discussion/decision-record.md:218`, and D4's rationale at line 91.

**Issue:** D4's rationale and R3 say the pair's git-ignored `.env` already binds the markers of the marker-carrying denied keys.
That holds: every one of them is per-worktree (reed's five, `shuttle.claude`, `shuttle.run_dir`), and no hub-wide denied key carries a marker in `internal/darnengine/template.yaml` or `internal/landingshed/template.yaml`.
But `envsource.Build` (`internal/envsource/envsource.go:37-47`) overlays `os.Environ()` on `.env`, with OS values winning.
`serverSpawnArgv` (`internal/reedengine/lifecycle.go`) starts the hub's shared tmux server from the booting process, and only `LYX_TRACE_ID` is stripped from its environment.
So a `LYX_SHUTTLE_CLAUDE` or `LYX_REED_SHELL` set in the environment of whichever pair's process boots the server is inherited by every pane of every pair, and through those panes by every `lyx` load there.
That route crosses pairs, which `.env` does not.
D4's decision stands: the local layer still must not add a route.
But R3 and Q12 hand the parent a narrower picture of today's untracked routes than the code shows.

## Rejected

- Focus 1, per key: every open key of each per-worktree module was checked against the bound.
  - reed: `width`, `height`, `collapsed_rows`, `min_full_rows`, `selvage.height_rows`, `segment_colors`, `watchdog` and `operator_quiet_s` act on the pair's own session.
    `width` and `height` go into `new-session -x/-y` for that session only.
    `operator_quiet_s` is floored to 5 when not positive (`internal/reedengine/io.go:244`).
    The server-global `set-option -g` pins use no config value.
  - shuttle: the open timeouts and poll keys fail or slow the pair's own runs.
    `startup_timeout_s: 0` zeroes the orphan sweep's age guard, but the sweep is confined to the pair's own run dirs while `run_dir` is denied.
    The prompt-cache TTL keys are tuning.
  - loom: `review_max_bounces` below 1 is refused by `loomrecipe.New`.
    The model keys, timeouts, `review_circling_checkpoint` (which escalates, never passes), `step_idle_timeout_min`, `friction`, `discussion_interactive` and `discussion_producer` are tuning or act on the writer.
    The exception is the fan keys, raised as F1.
  - webster: `self_fix_cap`, `master_timeout_min`, `recovery_timeout_min` and `verify_gate_attempts` are refused at 0 or below (`internal/websterengine/config.go:96`), so the verify gate cannot be switched off.
  - models: `engine` is a closed set and `defaults` keys are limited to `effort` and `version`, so an alias is a model pick.
  - batcher, orch, logger: batch sizing, the orch session's own tuning (`permission_mode` accepts `bypass` only), and per-anchor trace retention.
- Focus 2, the D6 strip: no defect.
  A worktree open map that differs from the default's in one entry is unequal as a unit and kept whole, so it resolves to the worktree's map as before.
  An equal one is stripped and resolves to the identical default map.
  Lists and scalars compare as leaves, `models` aliases as units, and burler lens strings as leaves, and each stripped value resolves from an equal value below.
  The acceptance row at line 237 pins the open-map case.
- Focus 3, the D4 premises: each stated premise holds against `internal/configengine/config.go:168` and `envsource.Build`.
  `.env` is read at the loading anchor, and it covers every marker-carrying denied key.
  The one gap is F2.
