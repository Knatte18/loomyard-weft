## Goal

`README.md` is rewritten from scratch as a teaching text for a reader new to LoomYard, and `docs/loomyard-overview.svg` is brought in line with the code.
The current README is kept unchanged as `README.old.md`.

## Scope

In scope:

- A new `README.md` that covers, in this order: what LoomYard is and the problem it solves, the core concepts (hub, worktree, board, task, run, orch, dobby), how a task goes from the board to a landed commit, how to get started, and links into `docs/` for detail.
- `docs/loomyard-overview.svg` corrected against the recipes and the dobby and landing code, and shown in the README where it helps.
- `docs/overview.md`'s one sentence about the figure, kept true.
- `README.old.md`, a byte-identical copy of today's `README.md`.

Out of scope:

- No Go code, recipe, stencil or `PATTERN.md` change.
- No rewrite of `docs/overview.md` beyond the figure sentence.
- No list of modules by their weaving names in the README.

## Decisions

### D1 — README is a teaching text in the order the board entry names

Decision: the README opens with what LoomYard is and why, then introduces hub, worktree, board, task, run, orch and dobby in the order a reader meets them, then follows one task from the board to a landed commit, then getting started, then links.
Rationale: the board entry fixes the order; each concept is defined before a later one uses it.

### D2 — Weaving terms appear only where needed, explained at first use

Decision: the README carries no module table and no list of weaving names.
A term such as batten, loom, shed or fabric appears only where the reader needs it for the task's path, with a plain-words gloss at that point.
Rationale: the board entry says the names tell a new reader nothing; `docs/overview.md` already holds the module map.

### D3 — Detail goes to `docs/` by link, never repeated

Decision: the README states the idea and links to `docs/overview.md`, the specs under `contracts/`, `PATTERN.md`, `docs/sandbox-howto.md` and `crucible/`; it repeats none of their detail.
Rationale: one place per fact; the README stays short enough to read in one sitting.

### D4 — Every claim is checked against the code and `lym help`

Decision: each command, recipe row, file name and behavior the README states is read from the recipes under `contracts/recipes/`, the package docs or `lym help` output before it is written.
A claim from the old README that no longer holds is corrected, and one about something that no longer exists is dropped.
Rationale: the board entry asks for it; the old README names stale things such as the orch running the batten passes.

### D5 — Dobby starts the runs, and the README says so

Decision: the README and the figure show a released board task being started by the dobby of the beam it belongs to, the prime's dobby for a task of the root, and the orch as the session that works the beam with the operator, not as the starter of runs.
Rationale: `internal/dobbyengine/doc.go` and `PATTERN-dobby-starts-runs` name dobby as the one that starts a task's run.

### D6 — The figure follows the current recipes

Decision: the figure's loom row lists the loom recipe's main line as it stands: Discussion, Plan, Webster, Parent-Align, the Code-Review perch, Describe, PR-Review, Finalize and Friction-Reflect, with the retired Publish, PR-Align and PR-Gate boxes removed.
The darn note says darn runs a writer, the same perch, Describe, PR-Review and Finalize.
The edge from the orch to the batten row is replaced by an edge from the board through dobby, and the box labels and the glossary are corrected to match, with dobby and beam added to the glossary.
Rationale: the board entry names the orch-starts-runs error and the changed landing rows; the recipes are the source.

### D7 — The README shows the figure once, and `docs/overview.md` keeps its link

Decision: the README embeds `docs/loomyard-overview.svg` once, at the point where a task's path is told, as a markdown image with alt text.
`docs/overview.md` keeps linking the figure, and its sentence describes the figure as it now is.
Rationale: the board entry says the README shows it where it helps the reader; one placement avoids repeating the picture.

### D8 — `README.old.md` is a byte-identical copy, made before the rewrite

Decision: `README.old.md` is created from `README.md` as committed on the branch with `cp`, before `README.md` is touched, and is never edited.
Rationale: the operator compares the two; any edit would defeat that.

## Constraints

- The PATTERN rules in the run prompt bind; this change touches no Go package, so none is in play beyond the prose rules for the docs.
- Prose follows the repo's semantic-line-break rule: one sentence per line in markdown, no fixed-column wrap.
- No count, list of current callers or other perishable specific is pinned in the README; it names the command or file that produces the fact.
- The SVG stays a self-contained file of the same size and style, valid XML.

## Auto-mode assumptions

- The README's audience is a developer who knows git and has used an AI coding assistant but has not seen LoomYard.
- "How to get started" covers the path in `lym help`: build, `lym fabric clone`, `lym beam start` or its shortcut `lym s`, then board entries.
- A pull request landing is described as the default the `pull_request` setting of `beam.yaml` gives, with the setting named, since the template value is read from `internal/dobbyengine/template.yaml`.

## Open risks

- The Getting-started steps are checked against `lym help` and the package docs, not run end to end on a fresh machine.
- The figure is edited as SVG text and checked as XML; it is not rendered by a browser in this session.

## Acceptance criteria

- D1: `README.md` has the sections in the order the board entry gives, and each concept is defined before a later section uses it.
- D2: `README.md` holds no table or list of modules by weaving name, and each weaving term it uses is glossed where it first appears.
- D3: `README.md` links to `docs/overview.md`, `contracts/`, `PATTERN.md`, `docs/sandbox-howto.md` and `crucible/` for detail and repeats none of it.
- D4: every command in `README.md` appears in `lym help` output, and every recipe row it names appears in a recipe under `contracts/recipes/`.
- D5: neither `README.md` nor the figure says the orch starts runs; both name dobby.
- D6: the figure names none of the rows Publish, PR-Align or PR-Gate, shows the current landing rows, and parses as XML.
- D7: `README.md` embeds `docs/loomyard-overview.svg` once, and `docs/overview.md` still links it.
- D8: `README.old.md` is byte-identical to the previous `README.md` (`git show HEAD:README.md` compared with `cmp`).
