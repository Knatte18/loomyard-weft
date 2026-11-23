---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:87
    summary: "D4 gives every refusal the one way forward `lym config reconcile`, which moves it, but that command moves only a worktree's own and local files; it does not reach a hub base file (needs --layer default), a profile block (moved only by D6's once-per-hub reinterpretation, which no reconcile verb reruns), a task's run_settings (moved only by D7's walk step), `--set` or a recipe reference (fixed only by the new name); Seed-Child also wraps a run_settings refusal in its own `lym board upsert` way forward, so the operator gets two. This breaks PATTERN-refusal-way-forward."
    fix: "In D4, replace the single way forward with one per site: a local or worktree file names `lym config reconcile`; a hub base file names `lym config reconcile --layer default` from the prime; a task's run_settings names `lym board upsert` with the new full name, and say Seed-Child's own wrapper carries that same text so only one way forward is printed; a profile block names editing profiles.yaml with `lym config profiles` to the new path; `lym config <module> --set` and a recipe setting reference name the new full name. Update the D4 acceptance criterion at line 186 to assert the per-site way forward, and the plan-writer note at line 209 and D5's bound at line 111 to match."
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:119
    summary: "D6 runs after configsync's FollowTemplate merge of profiles.yaml, so a shipped profile whose loom.code_max_bounces the operator edited (1 to 3) keeps code_max_bounces 3 as the operator's unit and also gains the template's code.max_bounces 1; D6's move then collides, and the shared configreg helper (D7) under D5's new-path-wins rule drops the operator's 3 for the template default, against the Goal that every existing value carries over."
    fix: "In D6, state the collision rule for profiles.yaml: either move the old keys of the file and of its template base (profiles.template-base) before the follow, inside configsync, so the follow sees the operator's value at the new path; or, when the reinterpretation finds both, keep the old key's value whenever it differs from the template base's value at the old key. Name which of the two in D6 and say the D7 helper takes that rule as a parameter if it differs from D5's. Add an acceptance criterion: a seeded profiles.yaml whose shipped solid.tapestry block the operator changed to loom code_max_bounces 3 holds loom code.max_bounces 3 after the walk."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:129
    summary: "D7 says a failed board read or write is retried by the next walk, but the failure is only logged, the walk still writes the build stamp, and the next walk comes only with the next build; no verb runs D7 on demand, so an old-name entry stays refused at Seed-Child for the rest of the build. D7 also does not say whether it runs when the hub-wide write was skipped because the board could not be pulled."
    fix: "In D7, either keep the build stamp stale when D7's board read or write fails (as a skipped hub-wide write does, so the next start verb retries), or state the bound plainly: a failed write waits for the next build, and Seed-Child's refusal names `lym board upsert` with the new name meanwhile (per F1). Also state that D7 does not run when the hub-wide write was skipped. Reword the D7 criterion at line 196 to the chosen retry point."
  - id: F4
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/discussion/decision-record.md:175
    summary: "The board entry asks that run_settings, profiles and `lym config --set` use the dotted full name, file included; the record reads that as the existing nested module object and `lym config loom --set discussion.review_ask_parent`, and puts the literal one-token form in Scope Out and Rejected alternatives. Whether the entry wants a one-token form is a question about what the task is for, which only the parent settles."
    fix: "Parent decision: confirm that the nested run_settings object and `lym config <module> --set <dotted path>` satisfy the entry's run_settings and --set bullet, or ask for a one-token form. The writer then drops the Open risks line and records the answer, or adds the form as a Decision."
  - id: F5
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:102
    summary: "When the hub base loom.yaml is absent, ReconcileHubWideAt seeds it from the prime's worktree copy through sparseSeed, which prunes units the template lacks; the prime file is moved only later by reconcileWorktree, so the prime's old keys are pruned from the seed instead of moved into it. D5 lists the files it moves but not the prime copy read as a seed."
    fix: "In D5, add that configsync applies the move table to the prime copy it reads as a seed before sparseSeed prunes it, and add the case to the D5 criterion: an absent hub base seeded from a prime holding code_max_bounces 2 holds code.max_bounces 2."
  - id: F6
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:176
    summary: "The mixed-build Open risk says an older build loads the moved hub base loom.yaml and refuses it; an older build refuses it only where a moved area name is an old bare scalar key (discussion, raddle, friction), while a moved code, split, review or pr map is only warned of as unknown and its tuned values silently read as template defaults."
    fix: "Reword the line-176 risk: an older build refuses a moved hub base that holds a discussion, raddle or friction map, and otherwise warns of the moved maps and runs on template defaults for every moved key, until it upgrades."
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:194
    summary: "The D6 criterion says an untouched seeded profiles.yaml equals the new template after the walk, but FollowTemplate re-encodes a changed file, drops blank lines between entries and copies template units without comments, so the file cannot equal the template byte for byte."
    fix: "Reword the D6 criterion to: after the walk, an untouched seeded profiles.yaml parses to the same YAML value as the new template."
---

### [BLOCKING] D4's one way forward reaches no fix at most of the sites it refuses

**Location:** `_lym/discussion/decision-record.md:87`

**Issue:** `lym config reconcile` with no layer runs `configsync.ReconcileAll` over the worktree's own and local files (`internal/configcli/configcli.go`, `runReconcile`).
The hub base file needs `--layer default`, whose `reconcileDefaultLayer` runs `ReconcileHubWideAt` alone and no reinterpretation (`internal/configcli/hubwide.go`), so a profile block written after D6's marker is never moved by any verb.
No reconcile verb touches board `run_settings`, and a `--set` or a recipe reference has nothing to move.
Seed-Child's `writeRunSettings` already wraps a `PrepareRunSettings` refusal in its own `lym board upsert` way forward (`internal/battenshed/seamchild.go`), so the operator would read two.
`PATTERN-refusal-way-forward` asks for a way forward a test reaches; four of the six sites fail it.
All four lenses raised this.

### [BLOCKING] D6 drops an operator's edit of a shipped profile

**Location:** `_lym/discussion/decision-record.md:119`

**Issue:** `profiles.yaml` is seed-only, and `ReconcileHubWideAt` merges it through `yamlengine.FollowTemplate`, which descends into mappings unit by unit (`internal/yamlengine/follow.go:41-60`).
The shipped template holds `loom: {code_max_bounces: 1, ...}` blocks (`internal/runprofile/template.yaml`).
An operator who edited one keeps the old key as their unit, and the follow adds the template's new `code` map, which file and base both lacked.
D6 runs afterwards and finds both names; D7 says D6 uses the same helper, and D5's rule keeps the new path, so the template default replaces the operator's value.
D6's criterion covers only an operator-added profile, which has no template counterpart and never collides.
Raised by interactions, existing-paths, bound-breaker and walkthrough.

### [MEDIUM] D7's retry happens only on the next build

**Location:** `_lym/discussion/decision-record.md:129`

**Issue:** `ensure` writes the stamp whenever `walkHub` returns not skipped (`internal/hubreconcile/walk.go`), and D7 only logs a failed read or write.
`carryBeamTaskSettings` shows the existing alternative: a marker left unwritten on failure, so the next walk retries.
D7 names neither that nor its behaviour when `reconcileHubWide` came back skipped.

### [MEDIUM] Whether run_settings and --set take a one-token dotted name is the parent's call

**Location:** `_lym/discussion/decision-record.md:175`

**Issue:** The entry's Wanted list says `run_settings`, profiles and `lym config --set` "use the dotted full name, file included".
The record's reading, where the module key or positional carries the file, is defensible, but it narrows what the entry literally asks for, and the record already defers it to the parent in Open risks.
Raising it as a `decision` finding routes it to the parent instead of leaving it as a risk line.

### [LOW] D5 does not move the prime copy read as a hub base seed

**Location:** `_lym/discussion/decision-record.md:102`

**Issue:** `ReconcileHubWideAt`'s absent-file branch seeds a per-worktree module's hub base from `primeConfig` through `sparseSeed`, which prunes to the template's units (`internal/configsync/configsync.go`).
`walkHub` reconciles the prime's own file only after `reconcileHubWide` (`internal/hubreconcile/walk.go`), so the seed still holds old names.
The values survive in the prime's worktree layer, so nothing is lost outright, but they are not in the seeded hub base.

### [LOW] The mixed-build risk overstates the older build's refusal

**Location:** `_lym/discussion/decision-record.md:176`

**Issue:** An older build's load warns of base-layer units its template lacks and reads template defaults (`internal/configengine/config.go`, `warnUnknownKeys`).
It refuses only a shape mismatch, which a moved bare area key (`discussion`, `raddle`, `friction`) causes and a moved `code`, `split`, `review` or `pr` map does not.

### [LOW] D6's criterion asks for byte equality that the follow cannot give

**Location:** `_lym/discussion/decision-record.md:194`

**Issue:** `FollowTemplate` re-encodes a changed file, drops blank lines between entries and copies a template unit without its comments (`internal/yamlengine/follow.go:53-57`).

## Rejected

- existing-paths F5 (`reviewbudget.CheckLocal` reads flat keys): D8's heading binds every reader to the nested path and its `reviewbudget` bullet changes the builders `CheckLocal` uses; naming each reader is call-site enumeration, which the rubric leaves to Plan.

## Focus notes

- Walk-order fact and D7's nesting claim: the Constraints' walk-order fact matches `internal/hubreconcile/walk.go` (`ReconcileHubWideAt` before `applyReinterpretations` inside one `BoltWrite`, worktree commits in `reconcileWorktree`), and D8's `FlagCodeAutoAccept` reading holds. D7's board write outside that write cannot nest with it. Not a defect.
- D4 against D1's table: the exact-path, shape-aware match leaves no case where an old name is admitted or a new path refused. Loom's `RetiredKeys` `pr_review` and `fan_review` prefix-match neither `pr_review_*` nor `pr.review_*`. Not a defect.
- D8: the recipe reference already splits at its first dot (`internal/shedrecipe/entries_settings.go`), and the segment fallbacks and budget builders are decided, not deferred. Not a defect.
- Relocation: Scope, Decisions and Auto-mode assumptions carry no narrative that belongs in `support-log.md`.
- Acceptance criteria: every D-id appears, D2 and D3's prose checks open with `Review:`, and D6 and D7 match D1's table; F2 and F7 cover D6's gaps.
- Bug, task-filing and own-entry-edit bullets: none apply.
