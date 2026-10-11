---
format: 5
approved: true
language: go
---

# Plan: fixer-critical-of-findings

A Burler fixer weighs each finding before it acts: it checks the finding's premise and, separately, the change the review's `**Fix:**` line proposes.
Today `burler-step-3-fix.md` allows a dispute only for a factually false premise and says nothing about a wrong proposed change, so a fixer applies a bad prescription because it has no other sanctioned outcome.
The plan gives the fixer three dispositions in a fixed order (dispute on either of two grounds, fix with its own change, fix as proposed), lets the next round's reviewer answer a dispute's evidence, and makes the converged carry-over entry say a finding was fixed or disputed instead of asserting a fix.
The judge, the decision rule, the review format and the fixer-report's free-form shape are unchanged.

## Card Index

1 — fixer-dispositions — The fixer stencil states the three dispositions, both dispute grounds with the every-fix bound, and the fixed-differently mark; the orchestrator's Validate step checks premise and proposed change; the claims table, the pattern background and the overview follow.
2 — reviewer-answers-dispute — The prior-rounds block tells a reviewer that re-raises a disputed finding to answer the dispute's evidence in its Issue; the test assertion and the package doc follow.
3 — carry-over-fixed-or-disputed — The converged carry-over entry says each finding was fixed or disputed and that the fixer report says which; the const comment, the test row, the two loom stencil sentences and their two claim whys follow.

## Shared Decisions

### Decision: one-rule-many-sites

The disposition rule is written once, in `burler-step-3-fix.md` on Card 1.
The orchestrator's Validate step names its two checks and points at instruction 3 without stating the rule.
Every other site (the claims table, the pattern background, the overview line and the burlerengine package doc) states or pins that text rather than its own variant.
Cards 2 and 3 read the stencil as Card 1 left it.

### Decision: no-new-tests

Every change is pinned by extending the existing claims table, the existing prior-rounds assertion and the existing converged test row, per `PATTERN-test-economy`; no package's test count changes, so the test budget file is untouched.

## verify:

```
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
