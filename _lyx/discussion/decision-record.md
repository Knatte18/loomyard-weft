# Decision record: cli-surface

## Goal

The orch knows every `lyx` command it may use without reading `--help` first, every command's help reads the same way, and `lyx board intake` carries an issue's comments onto the board.
Concretely: `lyx help index` prints a generated one-line-per-command index of the running binary's command tree, the orch gets that index with its role file on every launch and reload, a test-enforced form governs `Use`, `Short` and `Long` for every command including the board payload keys, and an imported issue's comments land in its board entry.

## Scope

In:

- `lyx help index`: a `help` module that keeps cobra's `lyx help [command]` and adds `index`, with an `--audience` filter.
- An `audience` annotation on every leaf command, with a test that walks the command tree and refuses a missing or unknown value.
- The index injected into the orch's role file at start, adopt and every reload.
- Reed recording, on each strand, the absolute `lyx` path its pane prelude exported, so the watcher's render reads the orch pane's binary (D4).
- An operator skill in the `ly` plugin that loads the index into an operator's own session.
- One documented form for `Use`, `Short` and `Long`, enforced by a tree-walking test, and the rewrite of every command that breaks it: flags out of `Use`, one argument spelling, lower-case `Short`, `Long` under a byte cap with its rationale moved to the package `doc.go`.
- Board payload commands name their keys in `Use`; `[json-payload]` disappears; `set-deps` addresses by `slug` or `id` like the other addressing verbs.
- `lyx board intake import` appends each issue comment to the entry body; `intake list` shows a comment count; lyx's own pointer comment is never carried back in.
- `PATTERN.md`, its `PATTERN-cli-cobra` background, `docs/overview.md` and the new module's `doc.go`, all in the same commit as the code.

Out:

- An MCP server over lyx (board note `lyx-mcp` stays a note).
- Hiding `role` or `internal` commands from `--help`; the annotation changes only what the index prints by default.
- Changing which commands a spawned role's stencil names; stencils keep naming their few commands directly.
- Flag descriptions in the index.
- Any change to shuttle's `Send` or the watcher's reload steps, except that `startAutoReload` turns a role-file render failure into idle with the reason, as the other render sites do (D4).
- Carrying comments posted by `lyx board intake close` or any other GitHub content beyond the issue body and its comments.

## Decisions

### D1: `lyx help index` lives in a `help` module that replaces cobra's built-in help command

Decision: a new `internal/helpcli` package exposes `Command()` returning the `help` command, installed on the root through cobra's `SetHelpCommand`, so `lyx help [command]` keeps its cobra behaviour and `lyx help index` is a child verb.
`index` walks `cmd.Root()` in memory at run time, reads no files, caches nothing, and prints the index as text, so it always matches the running binary.
The renderer itself (`clihelp.RenderIndex` or equivalent) lives in `internal/clihelp`, the package every `*cli` package and `cmd/lyx` already import, so `orchcli` can call it too (D4).
Errors stay JSON per `PATTERN-cli-cobra`; the success output is text, like `--help` and the `--text` listings.
Bound: the replacement keeps what cobra's built-in help command does, and only that plus `index`.
It completes `lyx help <tab>` with command names, as the built-in's argument completion does, and `lyx help <command>` prints that command's help.
Its `Use` is `help [<command>...]`, since a topic is a command path of one or more words.
Bare `lyx help` prints the root's help, as the built-in does.
The topic is resolved word by word from the root, and the first word that names no child is refused with a JSON error containing `unknown subcommand` and naming that word, in place of the built-in's printed "Unknown help topic" and root usage.
So `lyx help no-such-topic` and `lyx help board bogus` are both refused; the built-in's `Find` would print the board group's help for the second.
`help` meets `cmd/lyx/clitree_test.go`'s walk as a runnable group: it is exempt by name from the bare-group check alone, because bare `lyx help` prints the root's help rather than its own children, and it passes the unknown-subcommand check, the audience test and the form test like any other command; `help` and `index` both carry `operator`.
`lyx --json` lists `help` as a module with its `index` child; `renderCmdJSON` stops skipping `help` by name and keeps skipping `completion`.

Rationale: the operator named `lyx help index`; cobra's help command is replaceable in one call; placing the renderer in `clihelp` creates no new dependency edge, and an engine never imports cobra.

### D2: index line form

Decision: a nested markdown list, two spaces per level, one line per command, `lyx` and the parent path implied by the nesting.

- A group line is `` `<name>` ``: its `Short`.
  A runnable group (D3) renders its group line in the leaf form below, with its own positionals and flags.
- A leaf line is `` `<name> <rest of Use>` ``, then, when the command has local non-hidden flags not already spelled in `Use`, their names in one backticked run (`` `--a --b` ``), then `: <Short>`.
  `--json`, `--verbose` and `--help` are never listed.
- A group appears only when at least one leaf under it is in the selected audience.
- The `board` group line ends with the payload legend from D7, in one sentence.
- The index skips cobra's `completion` subtree, and never prints a command that carries no `audience`, so it shows only commands the D3 test has classified.

Rationale: the nested form is the smallest that lets the orch call most simple commands with no `--help` first.

### D3: an `audience` annotation on every leaf command

Decision: `internal/clihelp` declares the annotation key (alongside `SkipStencilSeedAnnotation`) and its closed value set: `operator`, `role`, `internal`.

- Every invocable command carries exactly one value, set where the command is built.
  An invocable command is a leaf (one with no subcommands) or a runnable group: a group with its own run function whose `Use` names a positional or that has local non-hidden flags of its own, such as `lyx config [module]` with `--print` and `--set`.
  The rest of this record says "leaf" for both.
  A shared verb body mounted under several groups (the `shedverbs` bodies under `loom`, `shed` and `batten`) takes its value at each mount, since the audience differs per mount.
- A group that is not runnable carries none; its audience is the union of its leaves.
- A hidden command is `internal`.
- A test in `cmd/lyx` walks `newRoot()` and fails on a leaf or runnable group without the annotation, a value outside the set, a non-runnable group that carries one, or a hidden leaf that is not `internal`.
- `lyx help index` prints `operator` by default; `--audience role|internal` prints that audience instead, for inspection.
  Bound: the flag selects one audience at a time and nothing else; the annotation never affects `--help`, which lists every non-hidden command as today.
- First classification: the table in the board entry's Part 1, with the borderline diagnosis verbs `webster status`, `webster reset`, `shuttle state` and the `status` verbs of every module as `operator`.
  The implementer reviews each leaf once against who calls it; the classification is in the annotation itself, kept nowhere else.

Rationale: a closed set the test pins means a new command cannot skip the decision, and the orch is handed only commands it may run, while a person at a terminal still sees everything.

### D4: the orch gets the index through its role file

Decision: the orch role stencil gains a `## Commands` section with one marker, `{{.command_index}}`, and a sentence saying the index is complete and `lyx <command> --help` gives the details.
`RenderRoleFile` fills the marker from a told index string.
The index describes the binary the orch calls: the `lyx` whose absolute path reed's pane prelude put first on the orch pane's `PATH` and exported as `LYX_BIN` when it last launched the orch strand.
That is the binary of whichever process launched the pane: `lyx orch start` on a fresh launch, or the `lyx` that ran a reed resume relaunching the strand.
Reed records that path on the strand: `Strand` gains a persisted field holding the absolute path `launchStrandLocked` composed the prelude from, set at every launch and relaunch, so the record follows the pane.
`lyx orch start` and `--adopt` render the role file only on a fresh launch, where their own binary is the one the new pane's prelude names, and the `resume-context` hook verb runs inside the orch's pane and inherits the prelude; each runs as a fresh process of that binary, so each computes the string through `clihelp`'s renderer from its own `cmd.Root()`.
The watcher cannot: its long-lived `lyx orch watch` process outlives a deploy, and its executable is the one `lyx orch start` ran from when it spawned the watcher, which differs from the pane's when `start` only spawns a watcher for a live strand or when a reed resume relaunched the pane from another `lyx`.
It takes an index source from `orchcli` instead, which at each render reads the orch strand's recorded path through reed and runs `<path> help index`; the run is logged per `PATTERN-spawn-observability`.
A deploy that rebuilds that same path is picked up at the next render.
Bound: the source runs only the strand's recorded path, never a `PATH` lookup and never the watcher's own executable; when the strand has no recorded path (one recorded before the field existed), the path no longer exists or the run fails, the render fails and the watcher goes idle with the reason, as a missing stencil does, at all three render sites.
`startAutoReload`, which today returns a render error as a failed tick and so ends the watcher after the consecutive-tick-error limit, goes idle with the reason like `startClearing` and `reloadAfterCompaction`; that one change to its error handling is the only change to the watcher's reload steps.
No typing step, stencil or shuttle call is added.

Rationale: the role file is already re-read on every launch path, including the reload after a compaction, where the watcher skips the skills step on the assumption that a compaction keeps skills; a skill load would miss exactly the cycle the operator wants covered.
Shuttle's `Send` refuses multi-line text, so typing the index is not open anyway.
This departs from the board entry's "same mechanism as the skill loading", for those reasons.
`orchengine` receives a string and imports no cobra, per `PATTERN-cli-cobra`'s engine rule; the stencil stays read from the told directory per `PATTERN-stencil-ownership`, and asks for no skill load per `PATTERN-role-skills-typed`.

### D5: an operator skill loads the index into the operator's own sessions

Decision: `plugins/ly/skills/cli/SKILL.md`, skill `ly:cli`, with a short section on when to reach for the index and the `--help` pages, and the line `` !`lyx help index` `` so Claude Code runs the command when the skill loads.
It is not added to `orchSkills`, since the orch's role file already carries the index.
The plugin version stays `1.0.0`.

Rationale: the content is always current and costs nothing to maintain; loading it into the orch twice would double its context cost.

### D6: one form for `Use`

Decision: `Use` is `<name>`, then positional arguments, then required flags only.

- A required positional is `<arg>`, an optional one `[<arg>]`, a repeated one `<arg>...`, an optional repeated one `[<arg>...]`; the arg name is lower-case with hyphens.
- A flag appears in `Use` only when the command marks it required; it is written `--flag <value>`.
  Optional flags live in `Flags:` alone, so every command that spells an optional flag in `Use` is rewritten.
- A board payload is one `{…}` token (D7).
- `lyx loom goto [run-id]` and `lyx loom review approve [slug]` adopt the bracket form.

Enforced by a test in `cmd/lyx` that walks `newRoot()`.
It splits `Use` on whitespace, except that a brace-balanced `{…}` span, spaces and nested braces included, is one token.
The first token equals the command's name, every further token matches one of the forms above, and every `--flag` token names a flag the command marks required.
For a `{…}` token the form test checks only that its braces balance; the keys inside it are D7's coupling test's.

Rationale: one spelling makes the index even and `--help` list each flag once; the required-flag exception is the one the operator granted.

### D7: board payload keys in `Use`, the grammar stated once

Decision: no board command says `[json-payload]`.
Each payload command's `Use` names its keys as one JSON-like token, where `?` marks an optional key, `a|b` exactly one of the two, and `<task>` stands for `upsert`'s payload:

- `get {slug|id}`, `remove {slug|id}`, `promote {slug|id}`
- `set-status {slug|id, status}`
- `set-deps {slug|id, depends_on}`
- `upsert {slug, title?, brief?, body?, depends_on?, isolated?, status?, kind?, labels?, recipe?, priority?, short_name?}`
- `upsert-batch {tasks: [<task>]}`
- `merge {remove_slugs?, upsert: <task>, set_status?: {slug|id, status}}`
- `intake import {issue, slug|into, title?, brief?, labels?}`
- `intake close {issue, reason, completed?}`

The grammar (`?`, `|`, `<task>`, one JSON object as a single quoted argument, unknown keys refused) is stated once, in the `board` group's `Long`, and in one sentence on the index's `board` line.
A test in `internal/boardcli` holds each command's `Use` token equal to what its parser accepts, compared as three sets: the required keys, the optional (`?`) keys and the exclusive (`a|b`) pairs, nested objects included.
Each parser declares those three sets once and validates against them, so the test reads the same declaration the parser enforces; where the parser is in `boardengine` (the task fields of `upsert`), the engine exports its declaration for the test.
`set-deps` goes through the same lookup as `get`, so `id` works.
Bound: `set-deps` gains `id` addressing only; its key set is otherwise unchanged and still refuses unknown keys.

Rationale: the operator decided the payload form is the same for every board command, so it is stated once and each `Use` names its keys; the coupling test is what keeps a key added to one side from being missing on the other.

### D8: one form for `Short`

Decision: `Short` starts in lower case, is one line, has no final period, and says what the command does in terms of when to reach for it.
The test in `cmd/lyx` enforces the mechanical part (case, one line, no final period); the "when to reach for it" part is review discipline.
Every capitalised `Short` (the `board`, `stencil`, `quarry` and `ide` trees) and every group `Short` that only names what the module is (`lyx`, `board`, `fabric`) is rewritten.

Rationale: the index is only as even as the `Short` texts it prints.

### D9: `Long` under a byte cap, rationale in `doc.go`

Decision: `Long` holds what the command does, when to use it, what its arguments and flags mean, and one or two examples; design rationale, implementation detail and refusal narratives move to the package's `doc.go` or stay in the refusal message, which already names the way forward.
A group's `Long` does not restate its subcommands in prose, since cobra lists them.
The root is exempt: its `Long` keeps its "Available modules" list, which `TestCLITree_EveryCommand`'s "root.Long names every mounted module" subtest pins so a new module cannot be mounted without being named there.
The test in `cmd/lyx` caps `Long` at a byte length held as the test's one constant, set at 2,000 bytes.
Bound: the cap is the only mechanical check on `Long`; its content is review discipline.

Rationale: `lyx <command> --help | grep …` is the orch's usual lookup, and the largest pages are the bulk of what a full dump costs.

### D10: PATTERN and docs

Decision: the `PATTERN-cli-cobra` entry and its background grow the three obligations this task adds: every leaf carries an `audience`, `Use` and `Short` follow D6 to D8, and `Long` sits under the cap, all pinned by the `cmd/lyx` tree walk.
No new PATTERN entry.
`docs/overview.md` lists the `help` module and the index; `internal/helpcli/doc.go` holds the module doc and the index's design; `internal/orchengine/doc.go` notes the role file's `## Commands` section.
Rows in `cmd/lyx/testdata/test-budget.yaml` rise for every package whose new top-level tests pass its row.

Rationale: `CLAUDE.md` requires docs in the same commit, and the form is a refinement of the existing CLI invariant, not a new one.

### D11: intake carries issue comments

Decision:

- `selfreportengine.Issue` gains a comment count read from the issue's own API field, so `ListOpenIssues` makes no extra call.
  `intake list` prints it as `comments` on every issue.
- `selfreportengine` gains a paginated comment fetch for one issue, oldest first, returning each comment's body and creation time.
- `intake import` fetches the comments after `GetIssue` and before the board write, so every network call stays outside the board lock.
- `boardengine.InboxIssue` gains the comments; the body builder appends each after the issue body under `## Issue comment, <YYYY-MM-DD>` (UTC date), oldest first, for a new note; under a fold, where the issue already sits under `## From issue #N`, the comment heading is one level deeper.
- The pointer comment lyx itself posts on import ("Imported into the board as `<slug>`.") is recognised by its one form, declared as a constant the writer and the filter share, and dropped.
  Bound: a comment is dropped only when its whole body, trimmed, equals that form with any slug between the backticks; every other comment, one that quotes the pointer sentence inside other text included, is carried verbatim, author not recorded.

Rationale: the operator treats a comment as part of its issue; the count lets the orch see comments before it chooses `slug` or `into`; the filter keeps a re-import after a prune from carrying lyx's own pointer back in.

### Added after Discussion (parent, 2026-10-10): Watcher index binary

Decision: The watcher renders the index from its own recorded path; D4 claims that path equals the orch pane's LYX_BIN only on the StartRelaunch branch, and the StartSpawnWatcher and reed-resume mismatch is stated under Open risks.

Rationale: A rare mismatch between two installed lyx binaries is not worth new launch-path plumbing in this task.

### Added after Discussion (parent, 2026-10-10): Missing-binary bound at auto-reload

Decision: startAutoReload keeps its existing error path: a failed index render is a tick error, and the watcher exits after the consecutive-error limit; D4's bound states this for that site, and the acceptance criterion for a missing path covers it.

Rationale: Scope keeps the watcher's reload steps out; the record must state the real behaviour, not change it.

### Added after Discussion (parent, 2026-10-10): Root Long keeps its module list

Decision: D9's group-Long rule exempts the root: its module list stays, since TestCLITree_EveryCommand pins it.

Rationale: Keeps an existing test-pinned contract; the rule targets subgroup Long texts.

## Constraints

- `PATTERN-cli-cobra`: `help` is a module with `Command()` and `RunCLI`; the registration oracle in `cmd/lyx/registration_test.go` scans `AddCommand` arguments only, so the plan either mounts `helpcli.Command()` through `AddCommand` before `SetHelpCommand` names it, or extends the oracle to see `SetHelpCommand`.
  The engine side (`orchengine`) imports no cobra; it receives the index as a string.
- `PATTERN-stencil-ownership` and `PATTERN-parent-directive`: the role stencil is read from the told stencils directory and filled with `stencil.Fill`; its new marker is filled from a told value; it asks no agent to load a skill.
- `PATTERN-orch-pane-single-writer` and `PATTERN-shuttle-stop`: nothing new is typed into the orch; the index travels in the role file.
- `PATTERN-spawn-observability` and `PATTERN-told-geometry`: the watcher's index source runs in `orchcli` and logs its `help index` spawn of the strand's recorded absolute path; `orchengine` is told the source and resolves no binary.
- `PATTERN-pane-binary-resolution`: reed records the path from the same `panebin.go` chokepoint that composes the prelude, so the record and `LYX_BIN` cannot disagree; the watcher stays a detached daemon spawned from the executable path, and its index source runs the recorded path, never `PATH`.
- `PATTERN-github-auth` and `PATTERN-agent-filed-issues`: the comment fetch goes through `selfreportengine`'s existing `repoClient`; no new package calls GitHub.
- `PATTERN-test-economy`, `PATTERN-test-speed`: every new test is tier 1, in-process, through `run()` or the package's public surface; GitHub is an `httptest` server as in `inbox_test.go`.
- `PATTERN-refusal-way-forward`: the board module is not bound; the only new refusals (`--audience` outside the set) name the set in the message.
- `CLAUDE.md`: docs in the same commit; test-budget rows; semantic line breaks in every `.md`; the plugin stays at `1.0.0`; `PATTERN.md` changes in the same commit as the code.
- Every `Short`, `Long` and index line is written under `scribe:prose`: no empty intensifiers, no pinned tallies.

## Auto-mode assumptions

No operator was available; each pick below is the Discussion producer's own, recorded in the support log's question ledger.

- A1: the index goes into the role file (D4), not a typed skill load, departing from the board entry's wording for the reasons given there.
- A2: groups carry no `audience`; a group's audience is the union of its leaves (D3).
- A3: borderline diagnosis verbs (`webster status`, `webster reset`, `shuttle state`, every `status`) are `operator` (D3).
- A4: `upsert` spells its full key list in `Use`; `upsert-batch` and `merge` reference `<task>` (D7).
- A5: the `Long` cap is 2,000 bytes (D9).
- A6: the comment heading records the date only, not the author, matching the form the operator used by hand on #479 and #480 (D11).
- A7: the operator skill is named `ly:cli` (D5).
- A8: the `help` module keeps cobra's `lyx help [command]` behaviour rather than making `index` a root verb (D1).
- A9: the watcher fetches the index at each render from the absolute path reed recorded when it last launched the orch strand, rather than restarting on a binary change (D4).

## Open risks

- The `cmd/lyx` form test fails until every command is rewritten; the rewrite and the test land together, so no intermediate state is green.
- `Long` trimming is judgment: a page cut to the cap may lose a refusal narrative the refusal message does not yet carry; the implementer checks each cut page's refusals name their way forward.
- Role file growth of about 3.5k tokens per read is accepted by the operator (2026-10-10); a later audience reclassification changes that cost silently.
- The strand's recorded binary is new in round 3: an orch strand launched before the field existed has no recorded path, so its watcher's renders go idle with the reason until the orch is relaunched, and the plan states how that reason names the way forward (`lyx orch stop`, then `lyx orch start`).
- A hub whose stencils directory holds an operator-edited role stencil without the new marker renders no index; `stencilstore`'s hash check leaves such a file alone, so the plan says how the missing marker surfaces (a render error naming the stencil, as the one-line stencils already do).

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 3: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/discussion/round-3-review.md; fixer report: _lyx/reviews/discussion/round-3-fixer-report.md.
  - missing-binary-bound-auto-reload (class design, severity MEDIUM, open in rounds 3)
  - watcher-index-stale-after-deploy (class design, severity MEDIUM, open in rounds 1, 2, 3)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- `lyx help index` exits 0, prints the nested form of D2, names every `operator` leaf in `newRoot()`, names no `role` or `internal` leaf, and prints a group only when it has a leaf in the audience.
- `lyx help index --audience role` and `--audience internal` print that audience alone; a value outside the set is refused with a JSON error naming the set.
- `lyx help board` still prints the board group's help; bare `lyx help` prints the root's help; `lyx help <tab>` completes command names; `lyx help no-such-topic` and `lyx help board bogus` are each refused with a JSON error containing `unknown subcommand` and naming the unknown word.
- The `cmd/lyx` tree walk reaches `help` and `index`: both carry `operator`, `help`'s `Use` passes the form test, `lyx help bogus` passes the unknown-subcommand check, and only the bare-group check exempts `help`.
- `lyx --json` lists `help` with its `index` child and does not list `completion`.
- The index skips `completion` and prints no command without an `audience`; `lyx config`'s line carries `[<module>]` and its `--print --set` flags.
- A leaf line omits flags its `Use` already spells and never lists `--json`, `--verbose` or `--help`.
- The `board` index line carries the payload legend; every board payload command's `Use` token in the index matches D7.
- The audience test fails on a leaf without the annotation, a runnable group without one, an unknown value, a non-runnable group carrying one, and a hidden leaf not marked `internal`.
- The form test accepts every D7 `Use`, each `{…}` span read as one token; it fails on a `Use` whose first token is not the command name, an optional flag in `Use`, a `--flag` not marked required, a positional outside the `<arg>`/`[<arg>]`/`<arg>...`/`[<arg>...]`/`{…}` forms, a `{…}` token whose braces do not balance, a `Short` starting upper-case or ending in a period or spanning lines, and a `Long` over the cap.
- Every board payload command's `Use` token equals its parser's declaration in required keys, optional keys and exclusive pairs, and the coupling test fails when a `?` or `|` marker disagrees with the parser; `lyx board set-deps '{"id":N,"depends_on":[…]}'` works and `{"slug":…,"id":…}` is refused as elsewhere.
- The role file rendered by `lyx orch start`, by the watcher's reload render and by `resume-context` contains the `## Commands` section with the `operator` index; the watcher's render takes the index from its told source, which runs the orch strand's recorded path and not a `PATH` lookup, its own executable or its own command tree; a strand relaunched by a reed resume from another `lyx` records that `lyx`, and the watcher's next render runs it; a missing or nonexistent recorded path sends the watcher idle with the reason at each of `startClearing`, `reloadAfterCompaction` and `startAutoReload`, never a failed tick; a role stencil without the marker fails the render with the stencil's name.
- `lyx board intake list` prints a `comments` count per issue, from the list response alone (no per-issue call in the fake server's log).
- `lyx board intake import` with `slug` writes the body, then each comment under `## Issue comment, <date>` oldest first across pages; with `into` the comment headings sit one level under `## From issue #N`; a comment whose whole body is lyx's pointer form is absent from the body, and a comment quoting the pointer sentence inside other text is kept; a failed comment fetch writes nothing to the board.
- `lyx help index` runs under `cmd/lyx`'s sandbox coverage or is excluded with a reason (`PATTERN-sandbox-coverage`).
- `lyx gate test ./...` is green with the rewritten `Use`, `Short` and `Long` texts, and `lyx loom lint-comments` passes on the new Go comments.

## Notes for the plan writer

- Annotation key and value constants: next to `SkipStencilSeedAnnotation` in `internal/clihelp/annotations.go`.
- The tree walk and its `help`/`completion` skip: `walkCommands` in `cmd/lyx/clitree_test.go`; the new tests can share the walker; the walker's skip of `help` goes, and `bareProblems`'s caller exempts `help` by name, per D1.
- Watcher spawn: `spawnWatcherProcess` in `internal/orchcli/spawn.go` resolves the path through `os.Executable()`; the PATH prelude is `composePaneLaunchLine` in `internal/reedengine/panebin.go`, called from `launchStrandLocked` in `internal/reedengine/spawn.go`, which both add and the resume loop in `internal/reedengine/lifecycle.go` go through; the strand record is `Strand` in `internal/reedengine/state.go`.
- Watcher render error handling: `startAutoReload` in `internal/orchengine/watcher.go` returns its `RenderRoleFile` error, which `watchloop.go` counts against `maxConsecutiveTickErrors`.
- `--json` help skips `help` by name in `renderCmdJSON` (`internal/clihelp/jsonhelp.go`).
- Role file render sites: `internal/orchcli/start.go`, `internal/orchengine/resumecontext.go`, and three calls in `internal/orchengine/watcher.go`; the stencil is `contracts/stencils/orch/orch-template-role.md`, whose header comment says it carries no markers and must change.
- Required-flag detection: cobra records `MarkFlagRequired` as a flag annotation.
- Board payload parsing: `resolveLookup` in `internal/boardcli/cli.go` for the addressing verbs; `decodeIntakePayload` in `internal/boardcli/intake.go`; task fields are validated in `boardengine`'s `UpsertTask`.
- Comments: `toIssue` in `internal/selfreportengine/inbox.go` is where the count joins; `issueContent` and `foldIssue` in `internal/boardengine/intake.go` build the body; the pointer comment text is built in `intakeImport`.
- The existing operator skill to pattern the new one on: `plugins/ly/skills/board/SKILL.md`.
- The board entry's Part 1 table is the starting classification; `lyx reed switch`, the `webster` batch verbs, `burler await-review` and `fabric merge-stage` are the operator's named examples of `role` or `internal`.
