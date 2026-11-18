# Pilot: simplify the design of landingshed without changing behaviour

`landingshed-design-simplify` · Task · enhancement, landing · starting · Seed-Child

## Why

The shrink study (`docs/research/rewrite/shrink-estimate-2026-11-18/report.md`, step 3) read `landingshed` once
and found design that spreads code over every caller.
This task is the pilot for `design-simplify-callers`: it shows whether such estimates hold before the same work is aimed at `fabricengine`.

## Scope

Simplify, in `internal/landingshed` and its callers only, the spots the study names:

- one result type in place of the four-value return handled at about 60 call sites;
- near-identical helpers merged: stop-result wrappers, fixer report parsers, table builders, the PR helpers repeated across Finalize and PR-Review, repeated error wraps;
- single-caller or test-only seams and adapters removed, the `parentPuller` interface never used in production included;
- the lock-freed, requeue and rebase layer around the fixer designed away by running the fixer outside the locked pass,
  if the spec's invariants allow it; otherwise the report says why it stays.

`docs/research/rewrite/shrink-estimate-2026-11-18/landingshed-spec.md` states the behaviour to keep: its rules and invariants hold after the change.

Out of scope: comments, compatibility and migration code, removing features, rewriting the tests beyond what the changed signatures need.

## Constraints

- No behaviour change: the existing tests pass, changed only where a signature they call changes.
- A spot that turns out unsafe to simplify is left, and the report names it with the reason.

## Report

The landing summary gives production code lines before and after, per spot,
against the study's estimate of about 600 to 800 lines saved by these spots, and the test lines changed.
