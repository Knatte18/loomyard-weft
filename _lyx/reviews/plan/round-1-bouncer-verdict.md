---
verdict: APPROVED
rationale: "All eight round-1 findings are fixed in the current cards (card 3 bound names the check-to-restamp window, card 9 adds step 0, the in-flight check, the plan-fingerprint compare and a way-forward reaching test, card 7 adds the _lyx-entry, dirty and outside-repo bounds, card 5 names the report as the _lyx exception, card 1 selects forthcoming cards from the reloaded plan); D1-D6 and every Constraint are carried, every targeted package runs plain and -tags integration, and no new finding survives the Do-not-flag list."
---

The review returned `APPROVED` with six MEDIUM and two LOW findings, and the fixer report claims all eight fixed.
I judged the plan as it stands now, after the fixer's edits, and confirmed each fix in the card text:

- F1: card 3's `Bound:` now says no site restamps after a failed check, states that an edit landing between check and restamp is adopted with the rewrite, begun hashes included, and names that window per site.
  Checked against the code: begin-batch (`beginbatch.go:303`), record-batch (`recordbatch.go:601`, `:644`), `Run` (`runlevel.go:537`) and validate (`validate.go:254`) all restamp only on the path where their edit check passed.
- F2: card 9's stuck reason and refusal-spec row end with re-running `lyx webster run`, and a reaching test asserts the stuck reason of an unfixed and of a spent attempt names the pre-fix head, each fix commit, `git reset --hard`, verify-and-commit and the re-run.
- F3: card 9's step 0 checks `FixStarter` before anything is recorded, a render error is a failed attempt, and a test asserts a nil starter leaves `IntegrationFix` unrecorded.
- F4: card 7's path rule covers the `_lyx` entry itself, its `Bound:` states that commits cannot reach a plan directory or `_lyx` outside the repository, and card 9 step 5 compares the plan fingerprint around the strand.
- F5: card 5's prompt forbids writing under the plan directory and committing under `_lyx`, and names the report and friction note as the only `_lyx` writes; the render test asserts the exception.
- F6: card 9 ends an attempt found in flight as `failed` before the `OK` early return, with a test over an `OK` report.
- F7: card 7 refuses a dirty worktree through `dirty`.
  `_lyx` is excluded in the repository's `info/exclude`, so the strand's own report under `_lyx` does not trip it.
- F8: card 1 selects forthcoming cards from the reloaded plan or `plan`, never `current`.

The review has no `## Focus departures` section, so there is nothing to ratify or reject.
Its "Focus verdicts not raised as findings" holds: D5's "summary.md names the fix commits ... for the review rows below to judge" licenses card 8's Describe stencil edit, since that stencil is how summary.md's integration sections reach a reviewer.

Fidelity: D1 is in cards 1–2, D2 in card 2, D3 in card 3, D4 in card 4, D5 in cards 5 and 7–9, D6 in card 6.
The Refusal Way-Forward constraint is met by card 9's row and reaching test, the Stencil Ownership constraint by card 5, the lease constraint by card 9's phase split, and the docs constraint by the `doc.go` edits in cards 2, 3 and 9.
Card 10 carries the TEMPORARY deletion.

Verify coverage: every targeted Go package (`planglyph`, `websterengine`, `webstercli`, `contracts/stencils`) runs plain; the three with `integration`-tagged tests run with `-tags integration`; `smoke` is only vetted; `go test ./...` covers the rest.

Every new routing edge states its bound: the forthcoming exclusion (cards 1–2), the unprompted hash move (card 3), the widened self-fix scope under the existing cap (card 4), and the fix strand's spawn and commits (cards 7 and 9).
