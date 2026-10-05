---
format: 5
approved: true
language: go
---

# Plan: Bouncer review rounds escalate to the parent, never halt for a human

A review segment's Bouncer stops halting a run for a human on something the run's parent can settle.
The judge may rule `CIRCLING` only from a configured checkpoint round on, and only when a gating finding sits on a ledger key open in the judged round and an earlier one, which a Go guard checks.
A `CIRCLING` ruling and a review budget spent without convergence both escalate: the Bouncer writes a brief and a one-line parent notice, returns `awaiting` with `parent_notice`, and `ly-drive` relays the notice to `LYX_PARENT`, whose one-shot fork records design calls with the new `lyx loom decision add`, decides through the widened `lyx loom circling accept|continue`, and resumes with `lyx loom start`.
The checkpoint and the review budget move into `loom.yaml`, the `recurring-finding` anomaly is removed, and the loomcli smoke tier is fixed.

## Card Index

1 — ledger-finding-labels — ledger entries carry the finding's class and severity
2 — circling-evidence — Go computes each key's open rounds and the circling evidence
3 — loom-config-keys — loom.yaml gains review_circling_checkpoint and review_max_bounces
4 — review-budget-from-config — review segments take their bounce budget from loom.yaml, and the Bounces seam agrees with Shed
5 — shed-parent-notice — Shed carries parent_notice on an awaiting halt
6 — escalation-stencils — escalation brief and parent-notice stencils
7 — escalation-record — the escalation record files in the Bouncer run directory
8 — circling-verbs-either-cause — the circling verbs settle an escalation of either cause
9 — checkpoint-judging — checkpoint-rendered decision rule and the Go guard on CIRCLING
10 — bouncer-escalation — the Bouncer escalates CIRCLING and a spent budget to the parent
11 — burler-exempt-round — the round after a budget continue spends no budget
12 — decision-append — discussionparser appends a post-Discussion decision
13 — decision-add-verb — lyx loom decision add
14 — verb-in-stencils-and-drive — Plan-Write names decision add; ly-drive relays parent_notice
15 — drop-recurring-finding — no GitHub issue for a recurring ledger finding
16 — smoke-strand-matching — the loomcli smoke tier passes

## Shared Decisions

- Escalation cause vocabulary is exactly `circling` and `budget`, declared once as `EscalationCause` in `internal/shedadapters`; the cause is `budget` whenever the `Bounces` seam reports `count >= budget`, even on a `CIRCLING` verdict.
- Per-round escalation files sit beside the other round files: `round-<N>-escalation.md` (Go-owned `round`/`cause` frontmatter over the rendered brief) and `round-<N>-parent-notice.md`; the circling decision file gains `cause`, and a legacy file without one reads as `circling`.
- Gating, wherever the guard reads it, is the judge stencil's definition: `class: design` at `MEDIUM` or worse, or any `BLOCKING`.
- Brief and notice wording lives only in the two new Bouncer stencils; Go interpolates only paths, round, cause, segment and slug, and the notice only the slug and brief path.
- Every new refusal lands with its `refusal-spec.md` row and a reaching untagged test in the card that adds it.

## verify:

go build ./...
go vet -tags smoke ./internal/loomcli/
go test -tags integration ./...
