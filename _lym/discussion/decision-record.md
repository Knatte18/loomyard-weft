# Decision record — README shows each project's slug

## Goal

A reader of the board's `README.md` or of a project page can see each project's slug, age and owner, so they can name the project in a command without opening the store.

## Scope

- In: the project sections of `README.md` and the header of every project page, `project-<slug>.md`, rendered by `internal/boardengine/render.go`.
- In: the golden tests in `internal/boardengine/render_test.go`, the README paragraph of the board package doc in `internal/boardengine/doc.go`, and the board line of `docs/overview.md` where it describes the project sections.
- Out: every other table, the Projects bullet list on a project page, the `Under:` link, the root header, the store, `lym board list` and every other view.

## Decisions

### D1 — One facts line per project: slug, age, owner

A project's facts render as one line, `` `<slug>` · Age <age> · Owner <owner> ``.
The slug is a code span, the age is the project's created age in the board's compact age unit, and the owner is the project's effective owner as a code span, or `none recorded`.
The age part is left out when the project has no created date.
Rationale: one shared line keeps the README section and the project page identical, and it reads like the design docs' metadata line, which also opens with the slug as a code span.

### D2 — README section: the facts line directly under the heading

Each project section of the README carries the facts line between its `##` heading and its brief.
The heading itself does not change.
Rationale: the heading already holds the link and the member count, and a link text with a code span in it reads poorly; a line of its own keeps the heading short and the slug easy to copy.

### D3 — Project page: the facts line after the Under line, replacing the beam's Owner line

Every project page carries the facts line after its `Under:` line, before the brief.
A beam page keeps its `beam` line, but its separate `Owner:` line goes, since the facts line now states the owner on every project page.
Rationale: showing the owner twice on a beam page would be noise, and the facts line gives a non-beam page the owner it lacked.

### D4 — Age and owner join the slug, by the parent's scope addition

The board entry asks for the slug only; the parent, `ly:orch`, relayed an operator scope addition in creel message #64 that each project section and project page also show the project's age and owner.
D1 to D3 apply it.
Rationale: an `ly:orch` message is the operator's delegate, and the addition stays inside the same two places the entry names.

### Added after Discussion (parent, 2026-10-30): Project sections also show age and owner

Decision: Beside the slug, each project section in the README, and its project page, also shows the project's age and its owner.

Rationale: Operator, 2026-10-30: the reader needs age and owner of a project at a glance.

## Constraints

- `Render` stays a pure function, with section names and fixed text declared in `render.go`.
- The effective owner comes from the board view's existing owner lookup, never a new resolution path, per `PATTERN-owner-machine`.
- Tests stay `tier0` goldens in the existing table, per `PATTERN-test-speed`.

## Auto-mode assumptions

- The project's age is its created age alone, not the revised age the tables show beside it, since the scope addition names age only.
- Owner on a non-beam project means its effective owner, read through its parent chain up to the prime owner.

## Open risks

- An external reader keyed on a beam page's `Owner:` line would stop finding it; no Go code reads the rendered files.

## Acceptance criteria

- D1: a golden shows a project with a created date rendering `` `<slug>` · Age <age> · Owner <owner> ``, and a project without one rendering the line with no age part.
- D1: a project whose effective owner is unknown shows `Owner none recorded`.
- D2: the README goldens show the facts line under every project section heading and above the brief, with the heading unchanged.
- D3: the project page goldens show the facts line after `Under:` on beam and non-beam pages, and no `Owner:` line on a beam page.
- D4: the package doc in `doc.go` and the board line of `docs/overview.md` describe the facts line in both places.
- Every existing board render test still passes with only the facts-line and beam Owner-line changes in its goldens.
