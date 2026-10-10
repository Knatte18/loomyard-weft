# Decision record: cli-surface

## Goal

The orch knows every `lyx` command it may use without reading `--help` first, every command's help reads the same way, and `lyx board intake` carries an issue's comments onto the board.
Concretely: `lyx help index` prints a generated one-line-per-command index of the running binary's command tree, the orch gets that index with its role file on every launch and reload, a test-enforced form governs `Use`, `Short` and `Long` for every command including the board payload keys, and an imported issue's comments land in its board entry.

## Scope

In:

- `lyx help index`: a `help` module that keeps cobra's `lyx help [command]` and adds `index`, with an `--audience` filter.
- An `audience` annotation on every leaf command, with a test that walks the command tree and refuses a missing or unknown value.
- The index injected into the orch's role file at start, adopt and every reload.
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
- Any change to shuttle's `Send` or the watcher's reload steps.
- Carrying comments posted by `lyx board intake close` or any other GitHub content beyond the issue body and its comments.

## Decisions

### D1: `lyx help index` lives in a `help` module that replaces cobra's built-in help command

Decision: a new `internal/helpcli` package exposes `Command()` returning the `help` command, installed on the root through cobra's `SetHelpCommand`, so `lyx help [command]` keeps its cobra behaviour and `lyx help index` is a child verb.
`index` walks `cmd.Root()` in memory at run time, reads no files, caches nothing, and prints the index as text, so it always matches the running binary.
The renderer itself (`clihelp.RenderIndex` or equivalent) lives in `internal/clihelp`, the package every `*cli` package and `cmd/lyx` already import, so `orchcli` can call it too (D4).
Errors stay JSON per `PATTERN-cli-cobra`; the success output is text, like `--help` and the `--text` listings.

Rationale: the operator named `lyx help index`; cobra's help command is replaceable in one call; placing the renderer in `clihelp` creates no new dependency edge, and an engine never imports cobra.

### D2: index line form

Decision: a nested markdown list, two spaces per level, one line per command, `lyx` and the parent path implied by the nesting.

- A group line is `` `<name>` ``: its `Short`.
- A leaf line is `` `<name> <rest of Use>` ``, then, when the command has local non-hidden flags not already spelled in `Use`, their names in one backticked run (`` `--a --b` ``), then `: <Short>`.
  `--json`, `--verbose` and `--help` are never listed.
- A group appears only when at least one leaf under it is in the selected audience.
- The `board` group line ends with the payload legend from D7, in one sentence.

Rationale: the operator measured this form at about 3.5k tokens for the `operator` audience, 11 to 12 percent below the full-path form, and it lets the orch call most simple commands with no `--help` first.

### D3: an `audience` annotation on every leaf command

Decision: `internal/clihelp` declares the annotation key (alongside `SkipStencilSeedAnnotation`) and its closed value set: `operator`, `role`, `internal`.

- Every leaf command (one with no subcommands) carries exactly one value, set where the command is built.
  A shared verb body mounted under several groups (the `shedverbs` bodies under `loom`, `shed` and `batten`) takes its value at each mount, since the audience differs per mount.
- A group carries none; its audience is the union of its leaves.
- A hidden command is `internal`.
- A test in `cmd/lyx` walks `newRoot()` and fails on a leaf without the annotation, a value outside the set, a group that carries one, or a hidden leaf that is not `internal`.
- `lyx help index` prints `operator` by default; `--audience role|internal` prints that audience instead, for inspection.
  Bound: the flag selects one audience at a time and nothing else; the annotation never affects `--help`, which lists every non-hidden command as today.
- First classification: the table in the board entry's Part 1, with the borderline diagnosis verbs `webster status`, `webster reset`, `shuttle state` and the `status` verbs of every module as `operator`.
  The implementer reviews each leaf once against who calls it; the classification is in the annotation itself, kept nowhere else.

Rationale: a closed set the test pins means a new command cannot skip the decision, and the orch is handed only commands it may run, while a person at a terminal still sees everything.

### D4: the orch gets the index through its role file

Decision: the orch role stencil gains a `## Commands` section with one marker, `{{.command_index}}`, and a sentence saying the index is complete and `lyx <command> --help` gives the details.
`RenderRoleFile` fills the marker from a told index string; the three callers (`lyx orch start` and `--adopt` in `orchcli`, the watcher's reload render, and the `resume-context` hook verb) compute that string through `clihelp`'s renderer from `cmd.Root()` of the process they run in and hand it to `orchengine`.
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

- A required positional is `<arg>`, an optional one `[<arg>]`, a repeated one `<arg>...`; the arg name is lower-case with hyphens.
- A flag appears in `Use` only when the command marks it required; it is written `--flag <value>`.
  Optional flags live in `Flags:` alone, so the 13 commands that spell them today are rewritten.
- A board payload is one `{…}` token (D7).
- `lyx loom goto [run-id]` and `lyx loom review approve [slug]` adopt the bracket form.

Enforced by a test in `cmd/lyx` that walks `newRoot()`: the first token equals the command's name, every further token matches one of the forms above, and every `--flag` token names a flag the command marks required.

Rationale: one spelling makes the index even and `--help` list each flag once; the required-flag exception is the one the operator granted.

### D7: board payload keys in `Use`, the grammar stated once

Decision: no board command says `[json-payload]`.
Each payload command's `Use` names its keys as one JSON-like token, where `?` marks an optional key, `a|b` exactly one of the two, and `<task>` stands for `upsert`'s payload:

- `get {slug|id}`, `remove {slug|id}`, `promote {slug|id}`
- `set-status {slug|id, status}`
- `set-deps {slug|id, depends_on}`
- `upsert {slug, title?, brief?, body?, depends_on?, isolated?, status?, kind?, labels?, recipe?, priority?, short_name?}`
- `upsert-batch {tasks: [<task>]}`
- `merge {remove_slugs?, upsert?: <task>, set_status?: {slug|id, status}}` (optionality of each key taken from the parser)
- `intake import {issue, slug|into, title?, brief?, labels?}`
- `intake close {issue, reason, completed?}`

The grammar (`?`, `|`, `<task>`, one JSON object as a single quoted argument, unknown keys refused) is stated once, in the `board` group's `Long`, and in one sentence on the index's `board` line.
A test in `internal/boardcli` holds each command's `Use` key set equal to the key set its parser accepts; where the parser is in `boardengine` (the task fields of `upsert`), the engine exports that set for the test.
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
The test in `cmd/lyx` caps `Long` at a byte length held as the test's one constant, set at 2,000 bytes so the median page passes untouched and the pages the operator measured at 2.5 to 5.4 KB must shrink.
Bound: the cap is the only mechanical check on `Long`; its content is review discipline.

Rationale: `lyx <command> --help | grep …` is the orch's usual lookup, and the ten largest pages are the bulk of the 40k tokens a full dump costs.

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
  Bound: only that form is dropped; every other comment is carried verbatim, author not recorded.

Rationale: the operator treats a comment as part of its issue; the count lets the orch see comments before it chooses `slug` or `into`; the filter keeps a re-import after a prune from carrying lyx's own pointer back in.

## Constraints

- `PATTERN-cli-cobra`: `help` is a module with `Command()` and `RunCLI`; the registration oracle in `cmd/lyx/registration_test.go` scans `AddCommand` arguments only, so the plan either mounts `helpcli.Command()` through `AddCommand` before `SetHelpCommand` names it, or extends the oracle to see `SetHelpCommand`.
  The engine side (`orchengine`) imports no cobra; it receives the index as a string.
- `PATTERN-stencil-ownership` and `PATTERN-parent-directive`: the role stencil is read from the told stencils directory and filled with `stencil.Fill`; its new marker is filled from a told value; it asks no agent to load a skill.
- `PATTERN-orch-pane-single-writer` and `PATTERN-shuttle-stop`: nothing new is typed into the orch; the index travels in the role file.
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

## Open risks

- `SetHelpCommand` and the registration oracle: if the oracle stays `AddCommand`-only, a `help` module registered another way is invisible to it; the plan picks one of the two routes in the Constraints section.
- The `--json` help renderer skips a child named `help`; with `help` a real module carrying `index`, the plan decides whether `lyx --json` lists it.
- The `cmd/lyx` form test fails until every command is rewritten; the rewrite and the test land together, so no intermediate state is green.
- `Long` trimming is judgment: a page cut to the cap may lose a refusal narrative the refusal message does not yet carry; the implementer checks each cut page's refusals name their way forward.
- Role file growth of about 3.5k tokens per read is accepted by the operator (2026-10-10); a later audience reclassification changes that cost silently.
- The merge payload's optionality in D7 is read from the parser at plan time; the coupling test catches a mismatch.
- A hub whose stencils directory holds an operator-edited role stencil without the new marker renders no index; `stencilstore`'s hash check leaves such a file alone, so the plan says how the missing marker surfaces (a render error naming the stencil, as the one-line stencils already do).

## Acceptance criteria

- `lyx help index` exits 0, prints the nested form of D2, names every `operator` leaf in `newRoot()`, names no `role` or `internal` leaf, and prints a group only when it has a leaf in the audience.
- `lyx help index --audience role` and `--audience internal` print that audience alone; a value outside the set is refused with a JSON error naming the set.
- `lyx help board` still prints the board group's help.
- A leaf line omits flags its `Use` already spells and never lists `--json`, `--verbose` or `--help`.
- The `board` index line carries the payload legend; every board payload command's `Use` token in the index matches D7.
- The audience test fails on a leaf without the annotation, an unknown value, a group carrying one, and a hidden leaf not marked `internal`.
- The form test fails on a `Use` whose first token is not the command name, an optional flag in `Use`, a `--flag` not marked required, a positional outside the `<arg>`/`[<arg>]`/`<arg>...` forms, a `Short` starting upper-case or ending in a period or spanning lines, and a `Long` over the cap.
- Every board payload command's `Use` key set equals its parser's accepted set; `lyx board set-deps '{"id":N,"depends_on":[…]}'` works and `{"slug":…,"id":…}` is refused as elsewhere.
- The role file rendered by `lyx orch start`, by the watcher's reload render and by `resume-context` contains the `## Commands` section with the `operator` index; a role stencil without the marker fails the render with the stencil's name.
- `lyx board intake list` prints a `comments` count per issue, from the list response alone (no per-issue call in the fake server's log).
- `lyx board intake import` with `slug` writes the body, then each comment under `## Issue comment, <date>` oldest first across pages; with `into` the comment headings sit one level under `## From issue #N`; a comment in lyx's pointer form is absent from the body; a failed comment fetch writes nothing to the board.
- `lyx help index` runs under `cmd/lyx`'s sandbox coverage or is excluded with a reason (`PATTERN-sandbox-coverage`).
- `lyx gate test ./...` is green with the rewritten `Use`, `Short` and `Long` texts, and `lyx loom lint-comments` passes on the new Go comments.

## Notes for the plan writer

- Annotation key and value constants: next to `SkipStencilSeedAnnotation` in `internal/clihelp/annotations.go`.
- The tree walk and its `help`/`completion` skip: `walkCommands` in `cmd/lyx/clitree_test.go`; the new tests can share the walker, and the skip must let the new `help` module's `index` through.
- `--json` help skips `help` by name in `renderCmdJSON` (`internal/clihelp/jsonhelp.go`).
- Role file render sites: `internal/orchcli/start.go`, `internal/orchengine/resumecontext.go`, and three calls in `internal/orchengine/watcher.go`; the stencil is `contracts/stencils/orch/orch-template-role.md`, whose header comment says it carries no markers and must change.
- Required-flag detection: cobra records `MarkFlagRequired` as a flag annotation.
- Board payload parsing: `resolveLookup` in `internal/boardcli/cli.go` for the addressing verbs; `decodeIntakePayload` in `internal/boardcli/intake.go`; task fields are validated in `boardengine`'s `UpsertTask`.
- Comments: `toIssue` in `internal/selfreportengine/inbox.go` is where the count joins; `issueContent` and `foldIssue` in `internal/boardengine/intake.go` build the body; the pointer comment text is built in `intakeImport`.
- The existing operator skill to pattern the new one on: `plugins/ly/skills/board/SKILL.md`.
- The board entry's Part 1 table is the starting classification; `lyx reed switch`, the `webster` batch verbs, `burler await-review` and `fabric merge-stage` are the operator's named examples of `role` or `internal`.
