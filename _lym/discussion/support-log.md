# Support log: Board README lists draft tasks under Ready

## Interview

A darn run holds no interview: the scope comes from the board entry, imported from issue #873, and a coarse reading of `internal/boardengine`.

## Rejected alternatives

- Keep a draft task in its Ready or letter section and only mark it: the current behavior, which the issue reports as wrong, since Ready says the task can start now.
- Place a draft task in its dependency letter and only move a dependency-free one out of Ready: splits unreleased tasks over several sections and still files them where startable tasks are.
- Render Draft right after Ready: puts tasks that cannot start ahead of the dependency layers, which can start once their dependencies land.
- Drop the `draft` code span inside the Draft section: the span is also the only draft mark on Running rows and in project member tables, and a per-table switch adds a branch for no reader gain.

## Review rounds

_No rounds yet._

## Question ledger

- Should a draft task with open dependencies sit in Draft or its letter layer? Self-pick: Draft, since release comes first.
- Should an isolated draft task sit in Draft or Independent? Self-pick: Draft.
- Where does the Draft subsection go? Self-pick: last among the task subsections, omitted when empty.
- Does a dependency on a draft task count as one on a Ready task? Self-pick: yes, through the existing special-layer depth rule.
