---
format: 5
approved: false
language: go
first_card: 17
---

# Plan: Skills and stencils — every role gets its procedure from a stencil and its skills from lyx

Every role lyx spawns gets its whole procedure from a stencil, the orch included, and lyx itself loads that procedure and the skills the role needs on every path into a session.
Cards 1–16 built that and are in the code.
The pull request was rejected with two findings, and this generation fixes them.
The orch role stencil misstates the run loop that batten runs, and Finalize's landing (card 17).
New tests broke `PATTERN-test-economy`: per-phrase stencil tests move into the claims table and the recipe-blind scan, a constant-getter test and two delegation tests go, and two pairs of skip-cause tests become tables (cards 18–21).

## Card Index

17 — orch-role-run-loop — the orch role stencil describes the loop batten runs and what Finalize does
18 — driver-stencil-claims — the driver stencil's phrase checks join the claims table and the recipe-blind scan
19 — orch-prompt-tests — the orch stencil wording checks join the claims table, leaving one render table test
20 — drop-getter-delegation-tests — delete the constant-getter test and the two delegation tests
21 — skill-skip-tables — the two skill-skip causes become one table test on each surface

## Shared Decisions

### Decision: stencil wording lives in the claims table

A check that a stencil contains or omits a phrase is a claim row in `contracts/stencils/claims_test.go`'s `wordingClaims`, never a test of its own.
A scan over a Go-declared set (recipe names, loom rows) stays a scan, in the existing scan test of that stencil.

### Decision: the role stencil follows the recipes

The orch role stencil states only what the batten and loom recipes do: batten creates the pair, drives the run, resumes a decided child and tears the pair down, and Finalize lands by pushing to main and closing the pull request.

## verify:

go build ./...
go vet -tags 'integration smoke' ./...
go test ./...
go test -tags integration ./...
