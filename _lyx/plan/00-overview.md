---
format: 5
approved: true
language: go
---

# Plan: webster-recovery

A webster run today needs a hand edit or a human in three states: a re-begun batch whose fork landed nothing (#329), webster's own rewrite of a begun card (#330), and an integration regression (#332).
Cards 1–2 give dispatch validation a forthcoming-target set and one scoping rule for begin-batch, run entry and validate.
Card 3 makes every sanctioned plan rewrite move the begun cards' recorded hashes.
Card 4 teaches the fork to fix what its own card broke.
Cards 5–9 build the one-shot integration-fix strand: its prompt, its state record and reclaim, its commit check, its report and summary record, and the stage that drives it.
Card 10 deletes the CLAUDE.md workarounds these fixes retire.

## Card Index

1 — dispatch-forthcoming-targets — `ValidateDispatch` takes a forthcoming card set whose Create and Rename New targets stay excluded from the status check
2 — one-dispatch-scope — `DispatchScope` computes begun and forthcoming cards once for begin-batch, run entry and validate
3 — move-begun-card-hashes — sanctioned-rewrite restamps move begun cards' recorded hashes; rebaseline never does
4 — fork-fixes-own-breakage — the fork prompt counts a test its card broke as the card's gate failure under the self-fix cap
5 — integration-fix-prompt — new stencil and renderer for the integration-fix strand
6 — integration-fix-state — state.json records the one fix attempt, and run entry reclaims its strand
7 — fix-commit-check — Go accepts the strand's commits only when non-merge, outside the plan and `_lyx`, and reconciled with HEAD
8 — integration-fix-record — the integration report and summary.md record the fix attempt
9 — integration-fix-attempt — the integration stage spawns one fix strand on a regression, re-triages, and keeps done or escalates
10 — drop-temporary-workarounds — delete CLAUDE.md's TEMPORARY items for #329, #330 and #332

## Shared Decisions

- No change to fork report fields Master reads, to batching, or to sequencing.
- Every plan rewrite is re-baselined immediately after the call that performed it, before its findings are examined, and persisted even when the call fails; card 3 extends that same restamp.
- The state-mutation lease is never held across the fix strand's wait or a verify run.
- `websterengine` stays bound by the Told-Geometry Invariant (no `lyxcwd`, no fabric import) and the Lyxdirs Single-Declarer Invariant (`lyxdirs.LyxDirName`, never a `_lyx` literal in path code).
- Repairing state.json of runs begun before this lands is out of scope.

## verify:

go build ./... && go test ./internal/planglyph/... ./internal/websterengine/... ./internal/webstercli/... ./contracts/stencils/... && go test -tags integration ./internal/planglyph/... ./internal/websterengine/... ./internal/webstercli/... && go vet -tags smoke ./internal/webstercli/... && go test ./...
