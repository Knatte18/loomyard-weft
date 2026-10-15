## Goal

Ship the mechanism by which every spawned role that writes or reviews code loads the hub's language skills after its own skill list: the darn writer, the Webster master and recovery strand, the code-reviewing Burler seats and the merge conflict session.
The hub's config names those skills, so lyx hard-codes no language.
The darn writer also gains `scribe:testing`.
This hub starts loading `scribe:golang-comments` and `scribe:golang-testing` once the operator sets its value after landing (Open risks).

## Scope

In:

- A new hub-wide config module, `language`, with one key, `skills`: the ordered skill names every code role loads after its own.
- Appending that list to the spawn specs of the darn writer, the Webster master and recovery strand, the merge conflict session at Merge-In and at Finalize, and every seat of a Burler half whose recipe row marks it as a code round.
- A Burler profile key that marks a code round, set on the Webster, Merge and Darn review rows and their fix rows in `contracts/recipes/loom-recipe.yaml` and `contracts/recipes/darn-recipe.yaml`, and accepted by `lyx burler run`'s profile YAML.
- `scribe:testing` added to `darnSkills`.
- Doc updates: each touched package's `doc.go` where it names its skill list, the new package's `doc.go`, `pattern/PATTERN-config-strictness.md`'s degrading-callers set, and `docs/overview.md` where it lists config modules.

Out:

- Every other spawned role keeps its skill list unchanged.
- Setting this hub's own `language.yaml` value: that file lives on the board, outside the task branch.
- Detecting a hub's language from its files.

## Decisions

### D1: A new hub-wide config module `language` holds the skill list

Decision: Register a config module named `language`, `HubWide: true` in `configreg`, its file at `<BoardDir>/_lyx/config/language.yaml`.
It carries one key, `skills`, a YAML list of skill names in load order, whose template default is the empty list.
A new package owns the module's template, `ConfigSpec` and loader; the plan names it.

Rationale: The language a hub's code is written in is a hub-level fact, which `PATTERN-hub-wide-config` puts in a `HubWide` module on the board.
Naming the module for the language, not for skills, lets the open toolchain question in `dotnet-scoped-verify` extend this module instead of adding a second hub-level language fact.
An empty default keeps a C# or Python hub from silently loading Go skills, as the board entry requires.

### D2: `skills` is classified Guard, closed to the local layer

Decision: The `skills` key's `ConfigSpec` category is Guard, so `LocalKeys` leaves it out and a worktree's local override cannot set it.

Rationale: `PATTERN-config-layers` counts "the prose or lens set a reviewer is given" as a guard, and the list reaches the code-reviewing Burler seats.
A pair that could drop or replace its reviewer's rules from an untracked file would hollow that review.

### D3: Loading degrades to an empty list, and only a hub file is read

Decision: The module's loader uses `configengine.LoadOrTemplate`, so a hub whose `language.yaml` is absent resolves `skills` to the empty list.
Wiring that knows no board dir, as in a standalone run, tells an empty list and never loads the module, so no worktree `_lyx/config/language.yaml` is ever read.
A present file that fails to parse, or an entry that is empty or carries leading or trailing whitespace, fails the load naming the key, and the message names `lyx config language` as the way forward.
Where that failure surfaces through a webster, shed or loom verb, it gets its row in `contracts/specs/refusal-spec.md` per `PATTERN-refusal-way-forward`.
The loader does not check that a named skill exists.
The new package joins the degrading callers in `pattern/PATTERN-config-strictness.md` in the same commit.

Rationale: A missing language file must never stop a code role from spawning; the role still runs with its own skills, as it does today.
Shuttle already skips an unknown skill with a logged warning and never fails or hangs the launch on one.
Bound: the degrade lets through only an absent hub file or an absent board, and either yields exactly today's skill lists.

### D4: Each code role appends the list after its own, skipping repeats

Decision: The wiring that builds a code role's engine or its deps loads the `language` module once per process and tells the engine the list.
The merge conflict session's engine is built inside `landingshed` at both Merge-In and Finalize, so `landingshed` loads no config;
`internal/loomcli`'s `wire()` loads the module beside `landing.yaml`, and the list reaches `landingshed.Deps` and then `mergeresolve.Deps` as a told value, with `landingDeps` staying free of I/O.
The engine sets the spawn spec's `Skills` to its own list followed by every language skill not already present, in config order, so a skill repeated in the config or already in the role's list appears once, at its first position.
The code roles are the darn writer (`darnSkills`), the Webster master and recovery strand (`roleSkills`), the merge conflict session (`conflictSkills`) and the Burler seats of a code round (D5).
The role lists stay package variables.
Every process that builds a code role's spawn loads the list itself, including `lyx webster recover-batch`, which spawns the recovery strand in its own process; an edit to `language.yaml` reaches the next spawn built by a process started after it, so a recovery strand can load a newer list than its master.

Rationale: Every code role here writes or reviews both production code and its tests, so each gets the whole list.
Webster's Agent-tool forks inherit the master's loaded skills, so they need no list of their own.
Loading once per process gives every spawn that process builds the same list.
Naming skills on the launch spec is the route `PATTERN-role-skills-typed` requires.

### D5: A Burler profile key marks a code round

Decision: `burlerengine.Profile` gains a boolean, set from a new recipe profile key `language-skills` on both `BurlerReview` and `BurlerFix` rows.
When it is true, every seat of that half loads `burlerSkills` followed by the told language list per D4; when false or absent, they load `burlerSkills` alone.
The key is set to true on the `Webster-Burler-Review`, `Merge-Burler-Review` and `Darn-Burler-Review` rows and on each one's fix row.
`burlerRoundProfile` and `burlerFixProfile` accept the key, and `lyx burler run`'s profile YAML accepts it too.

Rationale: One `burlerengine` engine runs every review segment, and only the recipe row knows whether its target is code.
Bound: the key only adds skills to a seat's load turn; it changes no rubric, lens, fan or gate.

### D6: The darn writer gains `scribe:testing`

Decision: `darnSkills` becomes `scribe:prose`, `scribe:code-quality`, `scribe:testing`, before the language list.

Rationale: The darn writer writes tests, and Webster and Burler already load `scribe:testing`; the board entry asks for it.

### D7: The template steers the list away from build skills

Decision: The template's comment on `skills` names comment and testing skills as the expected content, and says a build skill such as `scribe:golang-build` or `scribe:csharp-build`, whose raw commands bypass the hub's own build and test route, belongs only in a hub that has no such route.
Code does not filter any skill name.

Rationale: `scribe:golang-build` tells an agent to run `go test ./...` and similar commands, which this hub's agents must run through `lyx gate test`, and a verify gate already runs the hub's build.
Which skills to list is the operator's call per hub, so the template guides it and the code stays language-neutral.
Bound: the agents' session deny (`rawGoCommandPattern` in `internal/shuttleengine/claudeengine/settings.go`) refuses a `go test`, `go build` or `go vet` whose arguments hold a word containing `...` or the word `all`, a `go test` whose `-tags` value names `tmux` or `llm`, and any command naming the gate-slot, strand-name or prebuilt-binary environment variable.
It is a guardrail on static text, so a run behind a script or `bash -c` passes too.
Every other command a build skill names, such as `goimports -w`, `golangci-lint run`, `go install …@latest`, a `go test` of one package such as `./internal/x`, and every `dotnet build` or `dotnet test`, runs unrefused and outside the gate slots.
What bounds the hatch is the template comment and the operator's per-hub choice; the verify gate still runs the hub's own build, so no gate is skipped.

## Constraints

- `PATTERN-hub-wide-config`: the `language` file is read and written only at `<BoardDir>/_lyx/config/` (D3).
- `PATTERN-config-layers`: the `skills` key is classified in the module's `ConfigSpec`, and `TestModules_EveryTemplateKeyClassified` covers it.
- `PATTERN-config-strictness`: the new loader is a degrading caller, recorded in its background file.
- `PATTERN-role-skills-typed`: skills reach a session only through the spawn spec's `Skills`; no stencil asks for a language skill.
- `PATTERN-refusal-way-forward`: D3's load failure names its way forward.
- `PATTERN-test-economy` and `PATTERN-test-speed`: extend the existing skill-list and config-registry tests where they exist, at unit tier.
- The module order in `configreg.Modules()` is alphabetical.

## Auto-mode assumptions

- D1, D4: one shared list for every code role, held in module `language`, key `skills`, with an empty template default.
- D5: Burler profile key `language-skills`, set on the code rows only.
- Scope: PR-Rework and the plan writer are not code roles, since they rewrite the plan, not code.
- Open risks: this hub's value is set by the operator after landing.

## Open risks

- Until the operator sets this hub's value, code roles load no language skill, as today.
  The hub-wide reconcile after deploy seeds `language.yaml` from the template; then run `lyx config language --set 'skills=[scribe:golang-comments, scribe:golang-testing]'` in the prime.
- An operator who lists a build skill in a hub that builds through `lyx gate` sends agents to run that skill's commands, which D7's Bound lists as unrefused where they are not module-wide or tier runs, outside the gate slots; D7's template comment is the only thing that steers the operator away from listing one.
- Each listed skill adds to every code role's context on every spawn, and a misspelled one is skipped at once with a logged warning, so its rules never reach the role and nothing else signals it.
- `burler-fixer-fork` plans to fork the fixer from the reviewer, which would inherit the reviewer's language skills; that design decides whether the fix rows keep `language-skills`.

## Acceptance criteria

- The `language` module is registered `HubWide` in alphabetical order, `skills` is classified Guard, and writing `skills` to the local layer is refused.
- An absent `language.yaml` loads as an empty list; a file listing skills loads them in file order; an empty entry or one with surrounding whitespace fails the load naming `skills` and `lyx config language`.
- Wiring with no board dir tells an empty list and reads no worktree `language.yaml`.
- With an empty language list, every role's `Skills` equals today's list, except the darn writer's, which gains `scribe:testing`.
- With a non-empty list, the darn writer, the Webster master, the Webster recovery strand and the merge conflict session at Merge-In and at Finalize each spawn with their own list followed by the language skills in config order.
- A skill repeated in the config, or already in a role's own list, appears once, at its first position.
- A `BurlerReview` or `BurlerFix` row with `language-skills: true` spawns every seat of its half with `burlerSkills` followed by the language list; a row without the key spawns `burlerSkills` alone.
- A non-boolean `language-skills` value on a recipe row or a `lyx burler run` profile fails construction naming the key.
- The Webster, Merge and Darn review rows and their fix rows carry `language-skills: true`; the Discussion and Plan review rows and their fix rows do not.
- Roles outside D4 keep their skill lists unchanged.

## Notes for the plan writer

- Role lists: `darnSkills` in `internal/darnengine/spec.go`, `roleSkills` in `internal/websterengine/roles.go`, `conflictSkills` in `internal/mergeresolve/spec.go`, `burlerSkills` in `internal/burlerengine/engine.go`.
- `mergeresolve.New` is called from `internal/landingshed/mergein.go` and `internal/landingshed/finalize.go`.
- `landingDeps` in `internal/loomcli/landingdeps.go` is called from `arm.go` and `sharedbootstrap.go`; both pass the list.
- Burler profile mapping lives in `internal/shedrecipe/entries_burler.go` (`burlerRoundProfile`, `burlerFixProfile`, with `configRejectUnknown`) and `internal/burlercli/run.go`.
- `TestModules_SeedOnlyAndHubWideFlags` in `internal/configreg/configreg_test.go` pins the set of hub-wide modules.
- No D4 role resumes a session, so no resume path needs the list.
