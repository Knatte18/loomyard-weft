# Warn on resume when a run's stencils changed since it started

A run can last hours or days, and the board's stencils can change during it.
Before this change, a resumed run's later rows rendered whatever prompts were installed at that point, and nothing told anyone the prompts had changed.
Now a run records each stencil's digest when it is first seeded.
When the run is resumed, lym compares those digests with the installed stencils and tells the parent orch which ones changed.

## What changes

- `stencilstore.Pins` returns the `BodyHash` of each installed registry stencil's board copy, keyed by name.
  `stencilstore.ChangedPins` returns, in name order, each pinned stencil whose board copy now has a different body or is gone.
- `shedrun` declares the run's tracked `stencil-pins.json` record under its run directory (`_lym/shed/<slug>/`).
  It has `StencilPinsFile`, `StencilPinsRel`, `WriteStencilPins` and `ReadStencilPins`.
  Both the write and the read go through `internal/state`.
- `seedAndCommitBootstrap` writes the pins on the first seed only, and commits them along with the seed.
  On a re-entry, the existing pins are left as they are.
- A successful `lym loom start` compares the pins with the installed stencils, and so does a `lym loom resume` that spawns a Go driver or wakes a parked one.
  When any pinned stencil changed, the success envelope lists them under `stencils_changed`.
  The parent orch also gets one notice that names them and points to `lym stencil diff <name>`.
  The notice goes through `orchcli.NotifyParent` behind a new `notifyParent` seam on `loomCLI`.
- The package docs of `stencilstore`, `shedrun` and `loomcli` describe the new record and the new check, and so do the `start` and `resume` help texts.

## Design decisions

- **The whole registry is pinned, not a per-recipe subset.**
  Nothing maps a recipe to the stencils its rows render, and engine blocks are pulled in by includes.
  The registry is a superset of what any run may render, so hashing all of it never misses a stencil.
- **The digest is the body hash.**
  It ignores the stamp banner, so a refresh that only changes the stamp or the writer keys does not count as a change.
  Only a change to the text the stencil parser reads does.
- **Only the stencils are pinned, not the deployed specs.**
  The task names stencils only.
- **Start checks as well as resume.**
  Batten resumes a child run with `lym loom start`, so a check only in `resume` would miss most resumes.
  A first start compares against pins it has just written, so it never reports anything.
- **A warning never blocks the run.**
  The check names the change and points to `lym stencil diff`.
  It does not refuse the run, which differs from Bernstein's refusal to resume after its spec digest changes.
  A failed read, hash, compare or post is logged at Warn, and the verb's outcome stays the same.
- **Runs seeded before this change have no pins, so they report nothing.**
  Pinning them at their next start would claim those stencils were the ones the run started with, which may be false.

## Tests

- `stencilstore.TestChangedPins` covers these cases: unchanged copies, an edited body, a banner-only restamp, a removed copy, and a stencil installed after the pins were taken.
- `shedrun.TestStencilPins_RoundTrip` covers reading an absent record and the write-and-read round trip.
- `loomcli.TestResumeVerb_NamesStencilsChangedSinceTheRunStarted` resumes a halted run seeded for the Go driver.
  It covers a run without pins, unchanged stencils, and an edited stencil, and checks both `stencils_changed` and the parent notice.
- `loomcli.TestSeedAndCommitBootstrap_PinsStencilsOnTheFirstSeedOnly` checks that the first seed pins every installed stencil.
  It also checks that a re-entry after a stencil changed keeps the first seed's pins.
