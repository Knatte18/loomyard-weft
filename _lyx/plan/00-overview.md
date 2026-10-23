---
format: 5
approved: true
language: go
---

# Plan: plan-card-rules

Four changes to the plan and its cards, so faults a reviewer or the operator meets today are caught or owned inside the run.
The plan gate projects `PATTERN.md` from the cards' fenced entry lines and refuses growth past the headroom line, and refuses a declared entry with no verbatim line (D1, D2).
A card can declare it only adds to a ref with `**Extends:**`, and one pair rule in `internal/planparser` decides dependency edges for Webster's batch-order check and `tools/plandag`, which first gains the `-additive-guess` measurement (D3 to D6).
A decision that amends an approved plan, recorded with `lyx loom decision add --amends-plan`, is owned by the Plan-Review fix loop, and the arbiter, the orch and Webster's plan-conflict halt route to it (D7, D8).
A Merge-In or Publish verify failure is traced through quarry's `EncloseAt` to the plan cards whose targets hold the failing members, and the Publish failure note names them (D9).

## Card Index

1 — plandag-additive-guess — plandag gains -additive-guess and reports critical path and speedup with and without it
2 — pattern-overview-projection — internal/pattern projects the overview from prescribed entry lines and removes
3 — planparser-pattern-entry-lines — planparser exports a card's fenced PATTERN entry lines
4 — plan-gate-pattern-checks — the plan gate adds pattern-headroom, pattern-removes-prescribed and pattern-line-unprescribed
5 — plan-gate-pattern-base-wiring — every plan gate and validate-plan read the overview at the merge base
6 — extends-label — planparser parses the Extends type label and its two checks
7 — extends-group-readers — planglyph's delete-order regions and batcher's boundaries read Extends groups
8 — card-pair-rule — one planparser pair rule decides dependency edges, used by Webster's batch-order check
9 — plandag-extends-mark — plandag draws E cells and uses the shared pair rule
10 — plan-review-extends-guidance — the plan-review rubric checks Extends and the plan writer is told when to use it
11 — amendment-record — the tracked amendment record and its reader, writer and settle rule
12 — settle-round-start — Plan-Review's settle record records the judged round's start
13 — decision-amends-plan — decision add --amends-plan writes and commits the amendment record
14 — settled-guard-amendment — the settled-segment guard admits a goto onto Plan-Burler-Review over an amendment
15 — plan-commit-removes-amendment — Plan-Bouncer's commit seam removes an amendment its round carried
16 — rework-amendment — Plan-Review's skip seam declines and PR-Rework's archive removes the record
17 — plan-review-answer-key — the Plan-Burler-Review instruction names the amendment record and one editable-card rule
18 — recipe-amendment-target — the amendment_target recipe key and its loomrecipe accessor
19 — arbiter-amendment-target — the arbiter prompt carries amendment_target and the stencil routes amendments
20 — orch-amendment-route — the orch stencil routes a plan amendment through --amends-plan, goto and resume
21 — webster-conflict-amendment-way-forward — Webster's plan-conflict recurrence names the amendment route
22 — failure-trace-record — the Publish failure record gains cards, later_range and untraced
23 — planglyph-enclose — planglyph wraps quarry's EncloseAt
24 — planparser-target-holds — planparser decides whether a card target holds a code location
25 — failuretrace-package — failuretrace maps a failed verify log to plan cards and commit ranges
26 — landing-trace-seam — Merge-In and Publish record the trace through a told seam
27 — publish-failure-note-trace — the Publish failure note names the traced cards, ranges and untraced locations

## Shared Decisions

### Decision: no-overview-growth

`PATTERN.md` is already past its headroom line, so no card prescribes an entry line or grows the overview; the glyph-conversion background file gains the `EncloseAt` bullet and the overview line stays as it is.

### Decision: docs-under-edit

The binary that validates this plan does not know `**Extends:**` yet, so the cards list the doc files they extend under `**Edit:**` or `**Prosa:**`.
