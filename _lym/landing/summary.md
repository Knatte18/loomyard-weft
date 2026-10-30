# Board: project sections and pages show the project's slug, age and owner

A project's section in `README.md` showed its handle and title but not its slug, so a reader could not tell which slug to name in a command.
Each project section, and each project page, now carries one facts line naming the project's slug, age and owner.

## What changes in the rendered board

- Every project section of `README.md` has a facts line between its `##` heading and its brief, for instance `` `ship` · Age 3d · Owner `box` ``.
  The heading itself is unchanged.
- Every project page, `project-<slug>.md`, has the same line after its `Under:` line, before the brief.
- The slug is a code span; the age is the project's created age in the board's compact age unit, left out when the project has no created date;
  the owner is the project's effective owner as a code span, its recorded owner for a beam else its parent's up to the prime owner, or `none recorded`.
- A beam page keeps its `beam` line but drops its separate `Owner:` line, since the facts line now states the owner on every project page.

The change is `projectFacts` in `internal/boardengine/render.go`, called from the README's project sections and from the project page header, with the goldens in `render_test.go`, the README description in `doc.go` and the board line in `docs/overview.md`.
Every other table, the Projects list on a project page, the `Under:` link and every other view are untouched.

## Design decisions

- Age and owner join the slug by the operator's scope addition, which the parent recorded as an `Added after Discussion` decision.
- The facts line sits on its own line rather than in the heading: the heading already holds the link and the member count, and a separate line keeps the slug easy to copy.
- Age is the created age alone, not the created-and-revised pair the tables show, since the addition names age only.
- Owner is the effective owner from the board view's existing owner lookup, so a non-beam project shows the machine that owns it through its parents.

## To check by hand

Not run by hand; the golden tests in `internal/boardengine/render_test.go` cover the line in both places, with and without an age and with no known owner.
Open a rendered `README.md` and a beam's project page on a live board and confirm the facts line reads well under the heading and under `Under:`.
