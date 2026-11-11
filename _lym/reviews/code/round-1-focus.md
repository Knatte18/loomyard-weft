---
round: 1
focus:
  - "internal/landingshed/verifygate.go and finalize.go: does the rerun happen once, on the same head, before any fixer spawns, and only for a test-run failure? Check what a repeated failure and a non-test failure (build, lint) do."
  - "internal/landingshed/finalize.go and contracts/specs/loom-final-summary-spec.md: is a failure that does not repeat recorded as flaky in the landing report, and does the landing continue? Check the report stays consistent with the spec change."
  - "internal/gateslot/runlog.go and internal/gatecli/counters.go: does the failure report name the failing test and its error lines when the runner printed them, and fall back correctly when nothing parses? Check the parsing of VSTest and other runner output for unhandled formats and truncation."
  - "Rerun scope: if the rerun covers only the failed tests or tier, check a test that fails only in combination (parallel race) is not misjudged as passing, and that a rerun failing differently is handled."
  - "Tests in landingshed, gateslot and gatecli: check each new test for duplicated coverage, and each added behavior for missing coverage."
---

Seed focus for round 1. The decision record carries only the imported board entry under `## Goal`; the measuring stick is its two expected behaviours.
