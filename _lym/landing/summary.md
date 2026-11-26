# PATTERN: a test never fakes another module of this repository

Adds `PATTERN-test-stand-ins` and an inventory of the existing fakes.
No test or production code changes; the sweep of the existing fakes is follow-up work.

## The rule

- A test never hand-writes another repository module's state, records, config or output.
  It drives the real module, or the test belongs in that module.
- A stand-in for something outside the repository replaces that thing alone, below the one module that reaches it.
  It answers only recordings of the real thing, and a contract test holds it to the real thing.
- Because no gate runs `tier3`, an LLM stand-in's recordings are re-recorded by the task that changes how an engine
  launches, types into or reads its provider.

## What it changes in standing patterns

- `PATTERN-test-speed` no longer says a behaviour testable with a fake is tested at `tier0` that way.
  A behaviour takes the lowest tier its real modules run in.
- `PATTERN-test-isolation` kits no longer cover fakes of repository modules; the existing ones last until swept.
- The testkit package doc and `docs/overview.md` follow.

## Report

`docs/research/no-fake-other-modules.md` summarizes, with one inventory per area under `docs/research/no-fake-other-modules/`.
Each stand-in found is marked `real`, `goes`, `own` or `external`.
The points to read before accepting the rule:

- Fakes across modules are the normal test style here, so most rows read `real`, and many of those tests move up a tier.
  The gate cost was not measured.
- `shuttlefake` is not a stand-in for an LLM session: it replaces `claudeengine`, `codexengine` and `reedengine`.
  The LLM stand-in the rule describes does not exist yet.
- No `tier3` test compares a live session with a recording.
- Some replacements wait on writers that do not exist, such as a plan writer in `planparser`.
- The inventory was classified from searches and excerpts, and no replacement was tried, so small literals may be missing.

## Design decisions for the reviewer

The decision record does not settle these.

- A module is a Go package.
- The rule admits: a scripted implementation of an interface many modules fill, in the declaring module's tests;
  a value or file the code never interprets; clocks, sleepers, id sources and recording wrappers;
  config written by `configengine.Set`; removing another module's file or making it unreadable;
  and an agent-written file in its own parser's tests.
- It stays strict on values fed to pure functions, failures nothing can provoke, retired formats and version skew.
  The report lists these as open questions for the operator.
- No test enforces the rule; review does.

## Merge danger

**Door:** two-way
**Blast radius:** tests
