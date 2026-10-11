---
format: 5
approved: true
language: go
---

# Plan: gate-test-hygiene

Stop the `PATTERN.md` overview cap and the per-package test budget from blocking runs at Publish after a merge-in, or make the fix mechanical where blocking cannot be prevented: a headroom line below the cap, bound on growth by a lint beside the comment lint in the card gate and the round gate, with a self-check verb; a hard-cap finding and a budget failure that each name the value to fix; a one-time trim of the overview below the line.
Alongside, a Tier 1 gofmt guard with the tree brought gofmt-clean, and the last test-side `/proc` reads folded onto `tmuxkit` probes.

## Card Index

1 — gofmt-guard — add the gofmt guard to cmd/lyx and fix the six files it flags
2 — pattern-headroom — add the headroom line, the pure growth check and the bytes-over-cap message to internal/pattern
3 — lint-pattern-verb — add internal/patternlint and the lyx loom lint-pattern verb
4 — card-gate-lint-pattern — append lint-pattern to every webster card gate
5 — pair-merge-base-rename — rename darnMergeBase to pairMergeBase
6 — round-gate-headroom — run the headroom lint in the Webster-Burler round gate from the merge base, with the whole-diff form as a told flag
7 — test-budget-message — make the budget guard name the row value to set
8 — tmuxkit-proc-probes — add the parent-pid, state and comm probes to tmuxkit
9 — proc-reads-through-kit — route the six test-side /proc readers through the kit
10 — pattern-overview-trim — trim the overview to at most 15000 bytes, moving detail into background files

## Shared Decisions

### Decision: headroom-cut-is-the-smallest-passing-cut

- **Decision:** the bytes to cut a headroom finding names are the head size minus the larger of the base size and the line, never the head size minus the line alone.
- **Rationale:** this is the record's own fix of the Discussion-Review carry-over `headroom-byte-count-overstates-cut`, checked here against decision D2's wording; the row of `TestHeadroom` for a base above the line pins it, so a fixer is never told to cut wording the base brought in.
- **Applies to:** cards 2, 3, 6

### Decision: whole-diff-form-is-a-told-flag

- **Decision:** `loomshed.NewVerifyGate` is told its form by a `wholeDiff` flag, and a merge base reader means only the base of the headroom lint, or of both lints under the whole-diff form.
- **Rationale:** decision D3 tells the reader to every pair run's gate without switching a loom run to the whole-diff form, and today the reader's presence is what selects that form, so the two meanings must part; a flag on `shedrecipe.Env` set by darn's wiring alone keeps darn's behavior exactly as it is.
- **Applies to:** cards 5, 6

### Decision: patternlint-holds-the-git-reads

- **Decision:** the git and file reads behind the headroom lint live in a new `internal/patternlint`, shaped like `internal/commentlint`, and both the verb and the round gate call its `Lint`.
- **Rationale:** `internal/pattern` is a leaf that reads no git, and the verb in `loomcli` and the gate in `loomshed` would otherwise each carry the same base-mode logic; one package keeps the parity entry's pair a single function.
- **Applies to:** cards 3, 4, 6

## Rename mechanic

1. Run `git mv <old> <new>` FIRST, before any other change to the moved file.
2. Then make ONLY surgical edits (package declaration, imports, identifier
   retargeting) — no unrelated rewrites.
3. A genuinely new file with no predecessor belongs in a separate `Create` card, never folded
   into the `Rename` pair.
4. Never write the relocated file from scratch and delete the original — that loses
   git history exactly as an unstructured create+delete pair would.

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
