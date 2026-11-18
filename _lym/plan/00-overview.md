---
format: 5
approved: false
language: go
---

# Plan: result-level checks in Code-Review, Drift and PR-Review

Code-Review lets result-level issues reach PR-Review: widened test limits with no measured cause, departures on obsolete figures, and claims of agreement or improvement without before and after figures.
The plan adds generic result checks to the Code-Review rubric, a `Loosened-Check:` commit trailer every code-committing brief states, a Drift judge ruling on each loosened check in an existing test, and a PR-Review send-back to Code-Review for findings that are more than small, recorded as a friction note.
It touches the tapestry, darn and sampler recipes, the ones with a Code-Review segment.

## Card Index

1 — fabric-range-reads — list a range's changed files existing at its base, with diffs, and its whole commit messages
2 — drift-loosened-judgement — carry file rulings through the judge request, judgement, records and judge stencil
3 — drift-verify-loosened-checks — list, judge, store and halt on loosened checks in Drift-Verify, and render them in the verdict
4 — pr-review-small-max-setting — add the loom key pr_review_small_max, default 3
5 — pull-request-send-back-record — add the send-back mark, fix-here count and send-back count to the pull-request record
6 — stencil-plain-includes — let ReadForEngine resolve plain fragment includes, and read the PR-Review and sampler briefs through it
7 — bouncer-review-inputs — make the review input a list whose send-back file forces a full Code-Review generation, with its rubric note
8 — loom-pr-findings-verb — add lym loom pr-findings, deciding send-back or fix-here and recording the miss
9 — pr-review-send-back-row — reroute a marked send-back from PR-Review and clear it once the reroute persisted
10 — recipes-pr-review-reroute — give PR-Review on_reroute: Code-Bouncer in tapestry, darn and sampler
11 — loosened-check-fragment — add the shared Loosened-Check fragment and include it in every code-committing brief
12 — result-check-wording — add D1's rubric bullets, D4's PR-Review brief rule and D7's figures rule

## Shared Decisions

### Decision: no Go-written role commit today

- **Decision:** D2's clause "where Go writes a role's commit, the role's handoff carries the trailer lines and Go appends them" has no site to change.
  Every role commit of the three recipes is made by the session itself: Webster and Weaver forks, verify fixers, the darn writer, the sampler agent, the burler fixer, the gate fixers, the merge fixer and the PR-Review agent.
  Go writes only records commits, merge commits of a Parent-Align and the landing commit after review, none a role's code commit.
- **Rationale:** a search of the commit-writing code (`gitrepo`'s commit calls and their callers, `fabricengine.Merge`, `landingshed`'s landing merge) finds no Go-built message for a role's code change, so the criterion is carried by the shared fragment (card 11) alone.
- **Applies to:** cards 11

### Decision: the review base of Drift's listing

- **Decision:** Drift reads the same review base the Code-Review rubric's range uses, `taskMergeBase`, wired only where the origin record exists; a standalone run, which has no parent branch, lists no file.
- **Rationale:** D3 names that base, and a standalone run's Code-Review already reads its range as unknown.
- **Applies to:** card 3

### Decision: briefs reach the fragment through include-resolving reads

- **Decision:** `ReadForEngine` writes plain includes in place, so each code-committing brief includes the one fragment with a bare `{{template}}` action, never a Go-filled copy.
- **Rationale:** `PATTERN-stencil-ownership` keeps the fragment one stencil included by each brief; the seat and burler briefs already resolve includes through `ReadSet`.
- **Applies to:** cards 6, 11
