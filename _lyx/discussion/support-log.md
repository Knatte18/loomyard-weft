# Support log — sibling-overlap-check

## Interview

### Turn 1 — exploration (autonomous)

Read the board entry, notes `labels-from-diff` and `batten-ready-overlap`, `internal/boardengine` (`labels.go`, `board.go`, `store.go`, `config.go`), `internal/boardcli/cli.go` and `doc.go`, and `plugins/ly/skills/board/SKILL.md`.
Found: the vocabulary splits `types` from other labels; the envelope already has a `notice` field for successful writes with something to say; `set-deps` prints a bare ok; `upsert-batch` and `merge` take the upsert fields, `depends_on` included.
No operator answers in this mode; every pick below is an auto-pick recorded in the ledger.

### Turn 2 — design picks (autonomous)

Settled D1 to D5 of the decision record on the evidence above.

### Turn 3 — lens review by forks

Every fork returned a report.

- premises: `merge` target in `remove_slugs` counts every dependency as added — fixed, D1 merge baseline is the union of the target's and the removed entries' stored lists.
- premises: "Y has kind task" never filters, since only tasks carry `depends_on` — fixed, cut.
- premises: siblings must come from the post-write board so a removed entry is never one — fixed in D2 and a criterion.
- goal-scope: `upsert-batch` and `merge` exceed the entry's wording — rejected, already labelled an auto-pick with a reason tied to the goal.
- goal-scope / leanness (merged): the SKILL.md line-break fix sat as an implicit instruction in Notes — fixed, made explicit in D5.
- goal-scope: label-only writes stay silent and D3 adds the dependency and a way forward — rejected, both confirmations, no change needed.
- board-fit: overlap test should be one reusable engine function for `batten-ready-overlap` — fixed in D2.
- board-fit: `labels-from-diff` would feed this warning — fixed, clause in Open risks.
- pattern-fit: the no-warning cases belong in one table-driven test — fixed in Constraints.
- pattern-fit: the `set-deps` way forward must not be a denied command — rejected, the skill already directs agents to `set-deps`.
- pattern-fit: no test should pin the skill wording — fixed in Constraints.
- pattern-fit: the skill edit keeps semantic line breaks — fixed in Constraints.
- leanness: `PATTERN-refusal-way-forward` and `PATTERN-tolerant-records` bullets restate D3 — fixed, both cut, and D3's bound now names the record.
- leanness: Auto-mode assumptions restated picks — fixed, reduced to pointers, and the label-only consequence moved into D1.
- leanness: orch-copy said twice — fixed, kept in Scope Out only.
- leanness: Notes bullets name functions that Plan computes fresh — fixed, Notes cut.
- scenarios: an `upsert` omitting `depends_on` must stay silent — fixed, stated in D1 and a criterion.
- scenarios: `merge` into a new slug and `merge` removing Y were unspecified — fixed with the D1 baseline and criteria.
- scenarios: duplicate dependencies in a list — fixed, D3 names each shared dependency once, plus a criterion.
- scenarios: done dependency behaviour unstated — fixed, D1 excludes a done D, plus a criterion.
- scenarios: X and Z newly sharing two dependencies in one batch — rejected, D3's one line per pair naming every shared dependency already settles it.

## Rejected alternatives

- Refuse the write on a sibling overlap: the board entry says the write still succeeds.
- Warn on stderr: agents read the JSON envelope; `notice` already carries non-failing messages.
- A new `warnings` envelope key: duplicates `notice`.
- Warn on every dependency X holds, not only added ones: re-sending an unchanged list or editing a title would repeat old warnings.
- Count type labels as shared: `enhancement` is on most tasks and says nothing about code overlap.
- Exclude running siblings: a running task waiting on the same entry still collides at landing.
- Cover only `set-deps` and `upsert`: `upsert-batch` and `merge` write the same field and would reopen the gap.
- Compute siblings in `boardcli` from a separate read: a read outside the write lock can see a different board than the write did.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 — Which verbs warn? Auto-pick: `set-deps`, `upsert`, `upsert-batch`, `merge` (D4).
- Q2 — Which labels count? Auto-pick: module labels only, the non-`types` vocabulary (D1).
- Q3 — Which siblings count? Auto-pick: open tasks, running included, done excluded (D1).
- Q4 — Which dependencies trigger? Auto-pick: only those the write adds (D1).
- Q5 — Where does the warning go? Auto-pick: the envelope's `notice` field (D3).
- Q6 — Where is the check computed? Auto-pick: `boardengine`, inside the write's critical section (D2).
- Q7 — Batch pairs reported once or per member? Auto-pick: once (D3).
- Q8 — What is the "added" baseline for `merge`? Auto-pick: the union of the target's and the removed entries' stored lists (D1).
- Q9 — Does a done dependency trigger? Auto-pick: no (D1).
- Q10 — Fix the skill paragraph's missing line break? Auto-pick: yes, in the same edit (D5).
- Q11 — Does an abandoned sibling count? Auto-pick: no, a newly-ready start never starts it (D1).
- Q12 — Does a written task left `done` get siblings? Auto-pick: no, so the relation stays symmetric in a batch (D1).
