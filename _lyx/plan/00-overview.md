---
format: 5
approved: true
---

# Plan: test-tiers — cut the test load of a loom run

A Webster-Burler round runs only what its change can reach: a comment line-break lint, then an impacted-set command derived in Go from the diff since the last plan-verify pass, falling back to the full plan verify wherever it cannot tell.
Webster's gate, Publish and Finalize keep the full plan verify, and Publish adds the `tmux` tier through a repo-configured `publish_verify`.
`websterengine` and `loomshed` stop importing the code index, so most CLI test binaries stop linking tree-sitter, and two guards plus a test budget keep the suite from regrowing unseen.

## Card Index

1 — verifytree-per-command — per-command verified-tree records with their commit, and `LatestPass` for the round gate's base
2 — planindex-seam — cgo-free `planindex` package with the finding types and the `Index` interface; `planglyph` implements it
3 — websterengine-index-seam — `websterengine` takes the index on its geometry; the CLI layer wires it
4 — loomshed-index-seam — `loomshed`'s plan gates take the index; `shedrecipe.Env` carries it
5 — treesitter-link-guard — guard failing when a package outside the allowed list links tree-sitter
6 — comment-lint-engine — `commentlint` finds fixed-column wraps in the comment breaks a diff creates
7 — lint-comments-verb — `lyx loom lint-comments`, appended to every card gate; issue #421's line-break reading
8 — impact-set-engine — `impactset` derives the round command, its fallbacks and the `//lyx:guard` tests
9 — mark-guards — every repository-scanning test carries `//lyx:guard`
10 — round-gate — the Webster-Burler gate runs the lint, then the impacted-set command
11 — tmux-tier-green — the `tmux` tier is run on the clean tree; a failure whose cause sits in the `reedcli` smoke test files is fixed there, and any other stops the run before card 12
12 — publish-verify — landing config's `publish_verify`, run by Publish after the plan verify; loomyard sets the `tmux` tier
13 — plan-verify-guidance — plan spec, Plan-Write template and plan-review rubric drop the `tmux` tier from `## verify:`
14 — test-budget-guard — per-package top-level test budget, pinned at the after-state
15 — after-state — timing, tree-sitter link counts and link times in `docs/benchmarks/test-suite-timing.md`

## Shared Decisions

### Decision: the chain-cut seam

Measured on the before tree, 44 test binaries link tree-sitter under D8's tag set.
Nearly all reach it through two edges: `websterengine` → `planglyph` (via `configreg`, `hubgeom`, `standalonegeom` and `cliwire`) and `loomshed` → `planglyph` (via `shedrecipe` and the recipe wiring).
Cutting those two edges leaves the CLI packages that call the index (`cmd/lyx`, `loomcli`, `shedcli`, `webstercli`, `quarrycli`, `planglyph`) plus the test binaries of `websterengine` and `loomshed`, whose tests drive the real index.
The seam is `internal/planindex` (cgo-free types and interfaces), implemented by `planglyph.NewIndex` and wired at the CLI layer (cards 2–4).
A forced-relink `go test -c` costs about 0.5–1 s more for a binary that links tree-sitter; card 15 records the numbers.

### Decision: the round gate keeps the "verify" gate name

The Webster-Burler row is the only row naming the "verify" gate, so `loomshed.NewVerifyGate` changes in place (card 10) rather than adding a gate name; the recipe's row config is unchanged.

### Decision: binaries this run executes

The card and round gates of this run are rendered and run by the deployed `lyx`, so they pick up neither the lint verb nor the impacted-set command until the hub deploys this task.
The deployed binary loads this task's `landing.yaml` through `configengine.Load`, which ignores a key its template lacks, so the new `publish_verify` key does not break this run's Publish.

### Decision: the PR description

The PR description states the suite's top-level test count and the integration tier's wall time before and after (card 15).
It also states that `landing` config does not land, so the hub re-applies `publish_verify: go test -tags tmux ./...` to the parent's `_lyx/config/landing.yaml` after the landing, and that a task forked before then needs the key in its own copy for Publish to run the `tmux` tier.

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
