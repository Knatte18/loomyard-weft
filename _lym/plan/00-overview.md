---
format: 5
approved: false
language: go
---

# Plan: Keep a beam's pawl checklist beside the beam overview

While a beam's pawl run lives, `lym beam start`'s runs strand today replaces the `loom-status` checklist strand the pawl run's `lym loom start` put in the overview window, so the checklist is gone until a later `lym loom start`.
This plan shares one checklist strand spec between loom and orch, has the runs strand move the checklist into a window of its own instead of replacing it, lets reed name such a window by its strand's title, which is the run's recipe, and adds a line for the worktree's own run to the beam overview.

## Card Index

1 — checklist-strand-spec — loomengine holds the checklist strand's command, beside spec, sidecar path and seed recipe read
2 — loom-start-shared-spec — `lym loom start` builds the beside checklist from the shared spec, titled by the seed's recipe
3 — runs-strand-moves-checklist — the runs strand moves a `loom-status` strand to a window of its own instead of replacing it
4 — reed-title-names-own-window — reed names a window of its own by its strand's `Display.Title`, falling back to the role
5 — overview-own-run-line — the beam overview opens with a line naming the worktree's own run and its checklist window

## Shared Decisions

- The shared checklist pieces live in `internal/loomengine`, beside `LoomStatusStrandName`, because `internal/loomcli` imports `internal/orchcli` and the reverse import would cycle.
  `internal/reedengine`, `internal/reedengine/render` and `internal/shell` import no `internal/loomengine`, so the new imports add no cycle.
- Both callers read the recipe through `ChecklistRecipe`, `shedrun.ReadSeed` on `shedrun.SelfRunID` with no default, so a missing or unreadable seed gives an empty title, which reed reads as the role.
- Placement stays keyed on the runs strand's presence, never on a recipe name; the window name is read from the seed.
- `ensureRunsStrand` returns every warning it raised, since a failed move and a failed runs-strand add can happen in one bring-up, and never fails `start`.
- Reed's window naming is pinned at tier2 against real tmux; every other criterion is a tier0 test over pure functions or the existing fakes.
