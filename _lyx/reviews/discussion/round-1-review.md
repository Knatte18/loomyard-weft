---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md, Decisions, D4 (last Decision line and the third Rationale line)
    summary: D4 rests on a false premise that the Webster master and recovery strand are built in one invocation, but the recovery strand is spawned by a separate lyx webster recover-batch process, so they can load different lists and an edit does take effect inside a running loom run
    fix: In D4's Decision, replace the line "An edit to `language.yaml` takes effect at the next run invocation, never inside a running one." with "Every process that builds a code role's spawn loads the list itself, including `lyx webster recover-batch`, which spawns the recovery strand in its own process; an edit to `language.yaml` reaches the next spawn built by a process started after it, so a recovery strand can load a newer list than its master." In D4's Rationale, delete the sentence "Loading once per construction gives the master, the recovery strand and every Burler round of one invocation the same list." and replace it with "Loading once per construction gives every spawn one process builds the same list."
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md, Open risks, third bullet; _lyx/discussion/support-log.md, Interview, Turn 3, the scenarios misspelled-skill line
    summary: The Open risk says a misspelled skill costs every code spawn a second typed load turn, but shuttle skips a skill the session's skill listing does not name at once with no retry, which D3's own rationale also states
    fix: In decision-record.md's Open risks, replace the third bullet's clause "and a misspelled one costs every code spawn a second, typed load turn before shuttle skips it." with "and a misspelled one is skipped at once with a logged warning, so its rules never reach the role and nothing else signals it." In support-log.md's Interview Turn 3, rewrite the scenarios line about a misspelled skill so it says the skill is skipped with only a logged warning, in place of saying it costs a retry turn.
  - id: F3
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md, Decisions, D7; Open risks, second bullet
    summary: D7 lets any skill name through with no code filter but states no bound in the Decision, and the Open risk calls the template comment the only guard although a hub that builds through lyx gate refuses the raw build and test commands in its agents' session settings
    fix: Append to D7's Rationale a bound line in the form the other Decisions use, reading "a listed build skill can only steer a code role toward raw build and test commands; in a hub that builds through `lyx gate`, the agents' session settings refuse those commands and the verify gate still runs the hub's own build, so the cost is wasted turns, never a skipped gate." In Open risks, replace the second bullet's "D7's template comment is the only guard." with "each refused command costs a turn, and D7's template comment is the only thing that steers the operator away from listing one."
  - id: F4
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md, Acceptance criteria, the ninth bullet
    summary: The criterion excludes only the Discussion and Plan review rows, while D5 and Scope set the key on review rows and their fix rows, so the Discussion and Plan fix rows are left unpinned
    fix: Change the bullet's clause "the Discussion and Plan review rows do not." to "the Discussion and Plan review rows and their fix rows do not."
  - id: F5
    severity: NIT
    class: scope
    location: _lyx/discussion/decision-record.md, Constraints, the PATTERN-sandbox-coverage line
    summary: The line argues why a pattern does not apply rather than stating a constraint; it is deliberation the support log already records, and the plan writer needs nothing from it since a config module adds no top-level cobra command
    fix: Delete the Constraints bullet "`PATTERN-sandbox-coverage` binds lyx CLI modules; this task adds a config module and no CLI module." from decision-record.md; the support log's Turn 3 pattern-fit sandbox line already records the reasoning.
---

### [MEDIUM] D4's single-invocation premise does not hold for the recovery strand

**Location:** `_lyx/discussion/decision-record.md`, D4.

**Issue:** The master spawns from `websterengine.RunDeps`, built in `internal/loomcli/wiring.go` and `internal/webstercli/run.go`.
The recovery strand spawns from `websterengine.RecoverDeps`, built in `internal/webstercli/recoverbatch.go` and reached through `lyx webster recover-batch`, a separate process; the comment on `RunDeps.FrictionDir` in `internal/loomcli/wiring.go` states that the recovery prompts are composed in a separate `lyx webster` process.
So "once per construction" cannot give the master and the recovery strand the same list, and a `language.yaml` edit made while a loom run is in its Webster segment reaches that run's next recovery strand, contrary to "never inside a running one".
A plan writer taking the rationale at face value would look for a way to thread the master's list into the recovery spawn, which does not exist across processes.
D4's generic rule, "whatever constructs a code role's engine loads the module", is otherwise actionable for every construction site, including `burlerengine.New` in `internal/loomcli/wiring.go` and `internal/burlercli/wiring.go`; naming those sites is call-site enumeration this rubric does not ask for.

### [LOW] The misspelled-skill Open risk contradicts shuttle's skip behavior

**Location:** `_lyx/discussion/decision-record.md`, Open risks, third bullet.

**Issue:** `ClassifySkillLoad` in `internal/shuttleengine/claudeengine/skillload.go` classifies a skill as unknown when "the latest skill listing does not name it", and `settleLoadTurn` in `internal/shuttleengine/run.go` logs each unknown skill as skipped and returns only the missing ones for a retry.
A misspelled name is unknown, so it costs no second turn.
D3's rationale states the correct behavior, so the two sections disagree, and the real risk, that a typo silently drops a role's rules, goes unstated.

### [LOW] D7 lets any skill through without a stated bound

**Location:** `_lyx/discussion/decision-record.md`, D7 and Open risks.

**Issue:** D7 removes any code-side filter on skill names, which the discussion spec's bound rule covers, but the Decision states only what steers the operator, not what bounds the effect of a build skill once listed.
The bound exists: in a hub that builds through `lyx gate`, agents' session settings refuse raw module-wide `go` runs, and the verify gates still run the hub's build.
The Open risk's "only guard" wording understates that bound and leaves the Decision without one.

### [NIT] The criterion leaves the Discussion and Plan fix rows unpinned

**Location:** `_lyx/discussion/decision-record.md`, Acceptance criteria.

**Issue:** D5 sets `language-skills` on review rows and "each one's fix row"; the criterion pins the positive set with fix rows but the negative set without them, so a plan meeting the criterion could leave `Discussion-Burler-Fix` or `Plan-Burler-Fix` marked.
`contracts/recipes/loom-recipe.yaml` and `contracts/recipes/darn-recipe.yaml` hold five `BurlerReview` rows and five `BurlerFix` rows: Discussion, Plan, Webster, Merge and Darn.
D5's positive set covers every code round among them.

### [NIT] A non-applicability argument sits in Constraints

**Location:** `_lyx/discussion/decision-record.md`, Constraints.

**Issue:** `cmd/lyx/sandbox_coverage_test.go` collects top-level cobra commands only, and a config module is reached as `lyx config <module>`, so the line constrains nothing.
It carries no requirement the plan writer needs, which passes the completeness-before-leanness test for a cut; the support log already records the deliberation.

## Focus verdicts

- D4 and D5, who loads the list for Burler: D4's "whatever constructs a code role's engine" covers the Burler seats, whose engine is built by `burlerengine.New` in the loom and `lyx burler run` wirings, and D5 keeps the per-row switch on the profile while the list rides the engine; actionable, apart from the recovery-strand premise in F1.
- D2 against D3: no route remains.
  `skills` as Guard drops out of `LocalKeys`, so the local layer cannot set it; no-board wiring never loads the module, which closes `configengine`'s fallback to the worktree file that `PATTERN-config-strictness` describes; and `.env` only expands markers present in the hub file.
  D3 and D5 state their bounds; D2 tightens rather than relaxes.
  The one relaxing Decision without a bound is D7 (F3).
- D5 and the criteria: every code round is covered; the Discussion and Plan exclusion matches between D5 and Scope, and the criteria miss the fix rows (F4).
- D3 and D7: F3.
- Relocation: one Constraints line (F5); the Open risks lines each carry a risk or an operator step the record needs.
- Q8 and the hub-value step: not a defect.
  The Goal, the Auto-mode assumption and the Open risk agree that the operator sets the value after landing through `lyx config language --set` once hub reconcile seeds the file, which `internal/hubreconcile/walk.go` does with `SeedDefaults`; Q8 stays open in the ledger because the action is the operator's, not because the design is undecided.
