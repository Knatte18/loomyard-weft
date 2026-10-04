---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/10-board-model-docs.md (D13 paragraph)
    summary: Card 10 sends the D13 label list through the commit message body, but the Describe session that writes the PR description never reads commit messages, so the list does not reach the PR description
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/plan/04-board-load-migration.md (migrate.go bullets)
    summary: Card 4 adds two migration rules D12 does not word, removing a task's edge to a migrated note and defaulting an absent or unknown type to enhancement, without stating what licenses them
  - id: F3
    severity: LOW
    class: scope
    location: _lyx/plan/03-board-kind-and-labels.md (Board bullet on claimability)
    summary: The instruction to grep for every reader that gated on tier 1 is an open-ended hunt, and no code gates claimability on tier; the real sites are the render section meaning and CLI help
  - id: F4
    severity: LOW
    class: design
    location: _lyx/plan/09-board-intake-verbs.md (import bullet)
    summary: The import verb closes an inbox issue and writes the board but states no bound, while close states one
---

### [MEDIUM] D13 label list does not reach the PR description

**Location:** `_lyx/plan/10-board-model-docs.md`, the D13 paragraph.
**Issue:** The card puts the `board.yaml` snippet "in this card's commit message body ... so the PR description built from the branch carries it to the hub".
The PR title and body are the change description written by the landing Describe session (`internal/landingshed/describe.go`, `publish.go` through `summaryparser`).
Its prompt (`contracts/stencils/landing/landing-template-describe.md`) lists its sources: the decision record, `lyx board get <slug>`, the diff of the task branch against its merge base, and the run records.
A commit message is in none of them, and `git diff` does not show it.
D13 and the acceptance criterion require the exact list in the PR description, and the open risk it guards (every migrated entry with an area label refused on its next write after deploy) is the one the hub cannot see coming.
**Fix:** Keep the commit body as a record, and also put the derived list into a file the diff shows: a paragraph in `internal/boardengine/doc.go` (card 10's own file) naming loomyard's migrated area labels as a ready-to-paste `board.yaml` snippet, with the note that the hub removes it once applied.
State in the card why the commit body alone is not enough.

### [MEDIUM] Card 4 adds migration rules D12 does not word

**Location:** `_lyx/plan/04-board-load-migration.md`, the `migrate.go` bullets.
**Issue:** D12 lists the tier, type, bracket-prefix, note-dependency and `issues` rules and says no extra mechanism is wanted.
The card also (a) removes a migrated task's `depends_on` entry that names an entry migrated to a note, and (b) maps an absent or unknown `type` to `enhancement`.
Both are plausible: under D2 a task may depend only on a task, so (a) keeps the next write from failing D2, and under D3 a task needs one type label, so (b) keeps it from failing D3.
The card states neither derivation as its licence, so a reader sees silent data removal and an invented default.
**Fix:** In the card's Intent, state each rule as the consequence it is: (a) follows from D2 and the old tier rule, which admitted that edge only for a done dependency, and (b) from D3 and the old default type.
Say that nothing beyond these two is added.

### [LOW] Open-ended claimability grep in card 3

**Location:** `_lyx/plan/03-board-kind-and-labels.md`, "Only a task is claimable: grep the repository for every reader that gated on tier 1".
**Issue:** No production code gates claimability on tier.
The only tier readers are `layer.go`, `render.go`, `store.go`, `board.go`'s `PromoteNote` and `task.go`, all named by the card.
"Claimable" appears only as wording, in `render.go`'s section meaning and in the `promote` help.
An unbounded grep instruction invites invented scope.
**Fix:** Replace the sentence with a statement that no code gates claimability on tier, and that the claimable wording lives in the Tasks section meaning in `render.go` and the CLI help, both rewritten in this card.

### [LOW] Import verb has no stated bound

**Location:** `_lyx/plan/09-board-intake-verbs.md`, the `import` bullet.
**Issue:** `import` writes a note or a fold to the board and then closes an issue as completed.
The card states a `Bound:` for `close` only.
The rubric's attack-surface question applies to every new verb: what it can skip or let through, and what bounds it.
**Fix:** Add a `Bound:` to the `import` bullet: it acts only on an open, unrecorded issue of the inbox repo that is not a pull request, writes the board before touching GitHub, never deletes or edits the issue's content, and a recorded issue is a no-op.

## Focus verdicts

- Card 3 granularity: not a defect.
  The `Tier` and `Type` removal breaks every reader at once, and `validateLabels`, kind rules, `SetStatus` validation and the test fixtures share `validateWrite` and `Store.vocab`.
  A split would leave each half uncompilable or duplicate the fixture edits.
- Fidelity: D1 to D12 and every numbered orchestrator decision map to a card.
  The four named acceptance criteria are carried: `--body-file -` with stdin payload (card 6), `issues` duplicates on upsert (card 8), a fold adding no carried label and no type label to a note (card 8), `into` on a done entry or a task and `into` with `title`, `brief` or `labels` refused (card 8).
- Scope side, cards 7, 8, 9: no behavior beyond the decisions.
  The second-comment warning in card 7 and the `unknown keys refused` rule in card 9 follow from D10.
- Verify coverage: no gap.
  `go test ./...` runs every untagged test, including `tools/sandbox` and the docs link test, and the tagged line covers `boardengine/...` (`boardtest` included), `boardcli`, `selfreportengine`, `loomcli`, `battencli` and `cmd/lyx`.
- Attack-surface: `--body-file`, `intake close`, the nil-vocabulary store and the path-only `Board` each state a bound; `import` does not (F4).
- `ImpactSummary`: every `Edit` card states a conclusion, not a restatement of `Intent`.
- Card 4 and card 10 against D12 and D13: see F2 and F1.
