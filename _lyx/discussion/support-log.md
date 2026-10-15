## Interview

Autonomous run: no operator answered; every pick below is an auto-pick, and no advisors were seated.

### Turn 1 — exploration

Read the board entry and the related entries `llm-agnostic-engine`, `codex-engine`, `dotnet-verify-protocol` and `plugin-packaging`; none plans language skills or a language config, and none conflicts.
Read every Go-declared skill list, shuttle's skill loading (an unknown skill is skipped with a warning, never fatal), `configreg.Modules()`, the hub-wide and config-layer patterns, the strictness pattern, the Burler profile mapping and the recipe rows that run Burler.
Read the `scribe:golang-*` and `scribe:csharp-build` skills: the build skills tell an agent to run raw `go test ./...` or `dotnet test`.

### Turn 2 — questions and auto-picks

- Q1 Where does the list live? Recommended: a new hub-wide `language` module. Alternatives: a key in `gate`, or in per-worktree `shuttle`. Picked the recommendation (D1).
- Q2 Which roles get which skills? Recommended: one list for every code role. Alternative: per-purpose lists (comments, testing, build) mapped per role. Picked one list (D4), since every code role writes or reviews both code and tests, and no role should run a raw build skill (D7).
- Q3 How does Burler tell a code round from a Discussion or Plan round? Recommended: an explicit recipe profile key. Alternative: derive from `Profile.Segment`. Picked the key (D5).
- Q4 Template default? Recommended: empty. Alternative: the Go skills. Picked empty (D1), per the entry's no-hard-coded-Go requirement.
- Q5 Strict or degrading load? Recommended: degrading (D3).
- Q6 Local override? Recommended: closed, Guard (D2).
- Q7 Do PR-Rework and the plan writer count as code roles? Recommended: no; both write plans.

### Turn 3 — lens review by forks

All six forks returned.

- premises: the merge conflict session is built by `landingshed` at Merge-In and Finalize, not by CLI wiring — fixed (D4, Scope, Notes).
- premises, pattern-fit, scenarios (merged): no-board wiring would fall back to a worktree file — fixed: no board means an empty list and no read (D3, criterion).
- premises: the fix half has one seat — fixed: "every seat of that half" (D5).
- premises: the local refusal was unclear about which check it pins — fixed: the criterion pins the local-layer write refusal.
- premises: fork skill inheritance is provider-specific — rejected: true for the only engine; worded "Agent-tool forks".
- goal-scope: the goal is unmet until the operator sets the value — fixed: the Goal now states that the task ships the mechanism and the hub value follows.
- goal-scope: the Goal overstated "the `scribe:golang-*` rules" — fixed: names comments and testing.
- goal-scope: the Scope Out list was partial — fixed: "every other spawned role".
- goal-scope: the `lyx burler run` key goes beyond the entry — rejected: keeps the recipe and CLI profile vocabularies identical; listed in Scope.
- board-fit: overlap with `dotnet-scoped-verify` — fixed: the D1 rationale names it.
- board-fit: interaction with `burler-fixer-fork` — fixed: Open risks.
- board-fit: name C# in the template comment — fixed (D7).
- pattern-fit: `PATTERN-told-geometry` does not cover config, and "no engine loads config" was false — fixed: claim and Constraints line removed.
- pattern-fit, scenarios (merged): the load error had no way forward or load timing — fixed: names `lyx config language`, a refusal-spec row where applicable, loaded once per construction (D3, D4).
- pattern-fit: sandbox coverage was left as a check — fixed: the Constraints line says it binds CLI modules only.
- pattern-fit: the `--set` step may fail on an absent hub file — fixed: hub-wide reconcile seeds it (`hubreconcile` with `SeedDefaults`), noted in Open risks.
- pattern-fit, leanness: the `PATTERN-tolerant-records` line was padding — fixed: cut.
- scenarios: duplicates inside the list and surrounding whitespace — fixed (D3, D4, criteria).
- scenarios: a misspelled skill is skipped with only a logged warning, so its rules silently never reach the role — fixed: Open risks.
- scenarios: resume scenario — fixed: one line in Notes, since no D4 role resumes.
- leanness: rejected alternatives in the D1, D3, D4 and D5 rationales — fixed: cut.
- leanness: the auto-mode assumptions restated the decisions — fixed: they point at decision numbers.
- leanness: the gate-test criterion and the test-budget constraint restated repo rules — fixed: cut.

## Rejected alternatives

- Hard-coding Go skills in each role list: the entry forbids it, since lyx serves hubs in other languages.
- Per-purpose lists (`comments`, `testing`, `build`) with a role matrix in code: every code role would take the same comment and testing lists, and no role should take a build skill, so the split adds config surface with no role that differs.
- Per-role lists in config: duplicates one fact across roles, and an operator must keep them in step.
- A key in `gate.yaml`: gate configures slots for build and test runs, not the language agents write.
- A key in per-worktree `shuttle.yaml`: the language is a hub-level fact, which `PATTERN-hub-wide-config` puts on the board.
- Template default of Go skills: a C# hub would silently load Go rules.
- Deriving Burler's code rounds from `Profile.Segment`: ties behavior to a coloring field.
- Validating that each named skill exists: shuttle already skips an unknown skill with a warning.
- Strict loading: an absent file would stop every code role spawning.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 config home — resolved, auto-pick: new hub-wide `language` module (D1).
- Q2 role-to-skill mapping — resolved, auto-pick: one shared list for every code role (D4, D7).
- Q3 Burler code-round marker — resolved, auto-pick: `language-skills` profile key (D5).
- Q4 template default — resolved, auto-pick: empty list (D1).
- Q5 load strictness — resolved, auto-pick: degrading (D3).
- Q6 local override — resolved, auto-pick: closed, Guard (D2).
- Q7 PR-Rework and plan writer — resolved, auto-pick: not code roles (Scope).
- Q8 this hub's value — open for the operator: set after landing, see the record's Open risks.
- Q9 no-board wiring — resolved, auto-pick: empty list, no worktree read (D3).
- Q10 load timing — resolved, auto-pick: once per engine construction (D4).
